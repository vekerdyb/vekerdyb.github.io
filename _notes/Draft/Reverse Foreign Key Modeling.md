---
title : Modeling a reverse foreign key in Django
clickbait: Using 3 Tables Instead Of 2 Is Always Better For Indexing!
notetype : unfeed
date : 28-09-2021
---
## The problem
I needed to add a new property to an already existing table in our Django application with a PostgreSQL database. The deployment is automated using CircleCI. 
<!-- explain why a foreign key is needed --> 

For simplicity's sake, let's set up a somewhat contrived example. 

We run a chat service, and we have a `Message` model, already live, with hundreds of millions of messages stored. 

We now want to add a new model `PushNotificationToken` because we want to be able to send push notifications about some messages to the user. Each user can have many `PushNotificationToken`s, and each `Message` belongs to exactly one token. Users have been modelled on `Message` as a `ForeignKey`.

Normally I would go about it by adding a `ForeignKey`:
```python
class Message(models.Model):
	user = models.ForeignKey(User, on_delete=models.CASCADE)
    ... 
	
    token = models.ForeignKey(
		PushNotificationToken, 
		null=True,
		on_delete=models.SET_NULL,
	)
```

Let's create the migration:
```bash
./manage.py makemigrations
```

And we get something like this generated in our migration `0002`: 
```python
from django.db import migrations, models  
import django.db.models.deletion  
  
  
class Migration(migrations.Migration):  
  
    dependencies = [  
        ('chat', '0001_initial'),  
    ]  
  
    operations = [  
        migrations.AddField(  
            model_name='message',  
            name='token',  
            field=models.ForeignKey(null=True, on_delete=django.db.models.deletion.SET_NULL, to='chat.pushnotificationtoken'),  
        ),  
    ]
```

This is OK. (More on why it is not so good later). <!-- not good because of reversed dependency, and queries with 99% null values - all fields are included -->

Let's see the SQL migration created for this change:

```bash
$ ./manage.py sqlmigrate chat 0002

```

We get the generated SQL: 
```sql
BEGIN;

--
-- Add field push_notification_token to message
--

ALTER TABLE "chat_message" ADD COLUMN "push_notification_token_id" integer NULL CONSTRAINT "chat_message_push_notification_to_901191cc_fk_chat_pus" REFERENCES "chat_pushnotificationtoken"("id") DEFERRABLE INITIALLY DEFERRED; SET CONSTRAINTS "chat_message_push_notification_bi_901191cc_fk_chat_pus" IMMEDIATE;

CREATE INDEX "chat_message_push_notification_token_id_901191cc" ON "chat_message" ("push_notification_token_id");

COMMIT;
```

Note that this would lock the database until the entire `Message` table is indexed. 

From the [Postgres documentation](https://www.postgresql.org/docs/9.1/sql-createindex.html):
> Creating an index can interfere with regular operation of a database. Normally PostgreSQL locks the table to be indexed against writes and performs the entire index build with a single scan of the table. Other transactions can still read the table, but if they try to insert, update, or delete rows in the table they will block until the index build is finished. This could have a severe effect if the system is a live production database. Very large tables can take many hours to be indexed, and even for smaller tables, an index build can lock out writers for periods that are unacceptably long for a production system.

In our case this could potentially bring the production site for hours. 

The index creation can be separated out by implementing the field addition and indexing seprately. 

However it is worth noting another thing at this point: 
+ With the above data model we make the `Message` model dependent on push notifications. If we want to factor out push notifications to a separate service, we will have a hard time - but this is really just a mental exercise, maintainability is improved by keeping separate things separate <!-- improve english --> 
+ With each query on the `Message` by default we will select a column 99.99...% full of `NULL` values 

## A solution
Instead, we can create a `ManyToManyField` on `PushNotificationToken`, and create a through-table. To maintain the 1-to-many relationship we can constrain 


https://github.com/octoenergy/kraken-core/pull/36965

```
BEGIN;
--
-- Create model PushNotificationBindingMessage
--
CREATE TABLE "comms_pushnotificationbindingmessage" ("id" serial NOT NULL PRIMARY KEY, "message_id" integer NOT NULL UNIQUE, "push_notification_binding_id" integer NOT NULL);
--
-- Add field messages to pushnotificationbinding
--
ALTER TABLE "comms_pushnotificationbindingmessage" ADD CONSTRAINT "comms_pushnotificati_message_id_bdceeaa2_fk_comms_mes" FOREIGN KEY ("message_id") REFERENCES "comms_message" ("id") DEFERRABLE INITIALLY DEFERRED;
ALTER TABLE "comms_pushnotificationbindingmessage" ADD CONSTRAINT "comms_pushnotificati_push_notification_bi_10b2bc58_fk_comms_pus" FOREIGN KEY ("push_notification_binding_id") REFERENCES "comms_pushnotificationbinding" ("id") DEFERRABLE INITIALLY DEFERRED;
CREATE INDEX "comms_pushnotificationbind_push_notification_binding__10b2bc58" ON "comms_pushnotificationbindingmessage" ("push_notification_binding_id");
COMMIT;
```

The index is created on the small key