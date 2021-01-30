## Building a SaaS application with Django Web framework: The principle

A software which is running directly into the web browser and users have to pay for it on diverse way like per hour, or even per user, etc is called a Software as-a Service application (SaaS). Since those last years this model is very widely used by startup out there to sell their services. In this first article we are going to learn about the core principles of this.

## Ok let’s start, the principle.
Actually from my small experience talking about SaaS application design and infrastructure involve talking about data management, I mean questions like **how to store data in the database? how to access them? What kind of Database should be used? How to send an incoming request to the right application version?** and so on. Globally the concept is to allow the existence of many versions of the same application, each instance, should only show his data and manage only his clients.

## 1. So, How are data actually saved into the DB, what about accessing them?

In SaaS application each instance of the application is called a tenant, so the deal here is to choose a data separation technique, properly:

*A way to store tenant’s data by ensuring that they are saved without being mixed.*

Ok, actually this can be done directly into the database, usually there are 3 main ways to separate stored data, they can be called data isolation level within the database. So we have: Low isolation, semi-isolation and high isolation. Let dive into them before start coding.

- **Low isolation of data**
The principle here is to store tenant’s data within the same database and same tables. So you are in a case where each table has a column name for example, as tenant_uuid this column hold a unique identifier for a tenant, and there is a table in your database named for example as: tenants, it should store each tenant’s information like: name, uuid, registration_date, status, pricing_plan_id etc… In this case for all queries to the database, you will have to add a statement to ensure that you are retrieving right tenant’s data like:

```sql
SELECT * FROM 'table_name' WHERE tenant_uuid = 5
```

or with the Django's ORM

```python
TableName.objects.filter(tenant_uuid=5)
```
- **Semi-isolation of data**

The principle here is to save data within the **same database, same table BUT different schemas. **A schemas can be seen as a database into a database, it holds tables from as a normal database and you can create many schemas as you DBMS allow you, for those who use PostgreSQL, they use to work in the default schema named: public and SQL queries to table look sometimes like

```sql
SELECT * from public.table_name
```

With that said, semi isolation method consists in creating new schema for new tenants (your clients, the people who use your SaaS application), and all the created schemas contain the same tables. In this case the users’ data are not saved in the same place. They are technically in a separated table (or databases). So retrieving data with ORM is like:

```python3
TableName.objects.all()
# We assume that there is a tool ( a db router ) which is a charge of selecting the right schema to execute the query
```


we assume that there is a database router which is in charge to select the right schemas for the query’s execution. We’ll set it up in the next part of this article’s serie.

- **High data isolation level**

The principle here is to save tenant’s data within **different databases**. In this case when you have a new client you will just create a new empty database to store data. This is why we talk about high isolation, database can be even destroyed, but only one client will be affected, but it can be very heavy in terms of resource consumption. Accessing data with Django ORM can be done like:

```python3
TableName.objects.all().using('DB_NAME')
tableNameOject.save(using='DB_NAME')
```
the *using()* method (or parameter) is directly available in Django, by default its value is ‘default’; where to find it ? In the settings.py file, at the database information credentials :

```
DATABASES = {
    'default': {},
    'users': {
        'NAME': 'user_data',
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'mysql_user',
        'PASSWORD': 'superS3cret'
    },
    'customers': {
        'NAME': 'customer_data',
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'mysql_cust',
        'PASSWORD': 'veryPriv@ate'
    }
}
```

In the example above, there are 3 databases actually registered by their aliases :default, users, customers, so by default ( if you don’t specify using()) Django will choose default.

## 2. **How to know which tenant is required for a specific incoming request ?**

Incoming request represents here a user of your application who is connected via the browser and is about to use the application. The point here is to define what is called a **tenant detection strategy**;

It is a proper way to know how to interact with data stored in the database, it is not really related to the isolation level you have implemented in your application, the result of this operation is the name of the **schema** (in case of mid-isolation level) or the **tenant uuid** (in case of low isolation level) or **database** (in case of high isolation level) to use for data retrieval.

Usually to do that, developers use URL, by adding some indications within it, one of the most simplest and elegant way to do that IMO is by defining a sub-domain for each **tenant**; Let's assume that you application is a blog engine named www.bloghost.com each client will have a sub-domain and a url pointing directly to his blog like: client1.bloghost.com, another way is by defining url parameter as follow: www.bloghost.com/85DE5148WFE848 and this token can be a hash used to identify the **schema | tenant’s uuid | database ** related to the actual client’s data. This will totally depend on you.
Now when you have identified which tenant is aimed by a request you will just need to use the appropriate query to the database. You can even achieve this by using cookies. 

I won’t go too far in this article about that because the Django package we are going to use will handle this process for us, so I just tried to explain the concept.

Thank for reading this first part, it was just about knowing how it works and general terms used in this world. The next article will be more practical.
