---
title: "Changing Primary Key in Django Postgresql Setup"
date: 2019-02-25
categories: 
  - "tech"
---

After visiting my own code after 8 months to make a change, i realised that the schema of the table needed an update and will need a change in primary key field. What i had assumed would be unique turns out wasn't unique after all. So, i needed to put the default "id" field as auto incremental primary key.

Changing the primary key by raw sql in postgresql is simple. The tricky part is how you tell django that you have added a new field and changed the primary key so that it doesn't try to create a new field on running  "django manage.py makemigrations". This is where "state\_operations" comes for help.

By specifying state\_operations, you tell django that you have done "stuff" in raw sql that is equivalent to running "x operations" via orm migration.

Steps:

- Create an empty migration file: "python manage.py makemigrations <app name> --empty -n "change\_primary\_key"
- Put following code in the migration's operations: \[code="python"\] migrations.RunSQL( sql=\[ "ALTER TABLE $your table DROP CONSTRAINT $primary key constraint name", "ALTER TABLE $your table ADD COLUMN id SERIAL PRIMARY KEY;" "ALTER TABLE $your table ALTER COLUMN $old primary key DROP NOT NULL;" \], reverse\_sql=\[ "ALTER TABLE $your table DROP CONSTRAINT $primary key constraint name;", "ALTER TABLE $your table DROP COLUMN id", "ALTER TABLE $your table ALTER COLUMN $old primary key SET NOT NULL;", "ALTER TABLE $your table ADD PRIMARY KEY ($old primary key)", \], state\_operations=\[ migrations.RemoveIndex(model\_name='$model name as string', name='$primary key constraint name'), migrations.AlterField( model\_name='$model name as string', name='$old primary key', field=models.CharField(max\_length=100, null=True, blank=True) ), migrations.AddField( model\_name='$model name as string', name='id', field=models.AutoField(auto\_created=True, primary\_key=True, serialize=False, verbose\_name='ID')), \] ) \[/code\]
- The only change that needs to go in the model code for the table is: \[code="python"\] $old\_primary\_key = models.CharField(max\_length=100, null=True, blank=True) \[/code\]
- Now, if you run makemigrations, you should not find any new migrations created by django for changing primary key.

The best part about postgresql in this is that if you add a auto increment integer primary key, it auto-fills the rows with the keys starting from 1.
