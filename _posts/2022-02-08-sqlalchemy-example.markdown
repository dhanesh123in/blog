---
layout: post
title:  "Object-Relational Mapping with SQLAlchemy"
author: Dhanesh Padmanabhan
date:   2022-02-08
tags: SQLAlchemy ORM
---

Object-Relational Mapping (ORM) is a common approach for writing services interacting with relational databases in an object oriented fashion. Hibernate is a very popular ORM tool for Java. SQLAlchemy is the popular choice for Python. 

<h3>The Problem Statement</h3>

In this blog post, we will look at SQLAlchemy applied to a real life example where I had to create a database of members in the community I live in. A member here is defined as someone who belongs to a household that owns a property in the community. The data was being maintained in a spreadsheet where all the membership information was maintained for every property in our community. This spreadsheet also contained the history of how properties exchanged hands with new members coming in the system. 

<h3>The Table Definitions</h3>

I will present a simplified version of the problem to illustrate the cool features that SQLAlchemy provides. We will have 3 tables in our database for this example:

1. `members` - this will capture each member name, contact details and also have a mapping to the primary member in the household.
1. `properties` - this will consist of details of all properties in the community, with their description.
1. `memberships` - this will consist of the membership information for properties with membership start date and details of the primary member.

The code for defining these tables with all the entity relationships we need is given below:

```python
from sqlalchemy import Table, Column, Integer, String, DateTime, ForeignKey
from sqlalchemy.orm import relationship
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base

engine = create_engine('sqlite:///tst2.db', echo = False)

Base = declarative_base(engine)

class Members(Base):
    __tablename__='members'
    id=Column(Integer, primary_key = True)
    member_name=Column(String, nullable=False)
    mobile_number=Column(String, nullable=True)
    primary_member_id=Column(Integer, 
            ForeignKey('members.id', ondelete="CASCADE"))
    associated_members=relationship("Members", 
            cascade="all, delete, delete-orphan", 
            passive_deletes=False)
    associated_properties=relationship('Memberships', 
            back_populates="primary_member",
            cascade="all, delete, delete-orphan", 
            passive_deletes=False)

class Properties(Base):
    __tablename__="properties"
    id=Column(Integer,primary_key=True)
    property_id=Column(String, nullable=False)
    property_type=Column(String)
    property_desc=Column(String)
    completion_date=Column(DateTime)
    handover_date=Column(DateTime)
    membership_info=relationship("Memberships",
            back_populates="for_property", 
            uselist=False, 
            cascade="all, delete, delete-orphan",
            passive_deletes=False)

class Memberships(Base):
    __tablename__='memberships'
    id=Column(Integer,primary_key=True)
    pid=Column(Integer,ForeignKey('properties.id', 
            ondelete="CASCADE"))
    primary_member_id=Column(Integer,ForeignKey('members.id',
            ondelete="CASCADE"))
    membership_date=Column(DateTime)
    primary_member=relationship("Members", 
            back_populates="associated_properties")
    for_property=relationship("Properties", 
            back_populates="membership_info")
```

Ofcourse there is a lot going on here. We basically have three classes mapping to the three tables. Each class defines all the columns, their datatypes and the primary & foreign keys. There are also relationships defined in each of these classes, which I will describe in a bit. In this example, I have selected sqlite as the database.

To create these tables with these constraints, all we need to do is execute the following commands:

```python
Base.metadata.drop_all(engine)
Base.metadata.create_all()
```

<h3>Relationships - Deep Dive</h3>

Let's delve a bit deeper into the relationships before we do some example database operations. The variables that are defined as relationships return objects of associated objects of other table entities. For example, `Members.associated_members` will give a list of `Members` objects corresponding to the members associated with the primary member, and similarly `Properties.membership_info` will return the `Memberships` object corresponding to the property. 

The relationships work in tandem with a Foreign Key-Primary Key relationship. The relationship variables can be defined either on the class consisting the primary key or the class consisting the foreign key, or on both. The nature of relationship is one-to-many, or one-to-one when defined on the class with primary key, and it is many-to-one when defined on the table with foreign key. The associated relationships can be provided using `back_populates` option. The `useList` option can turned on or off to control the one-to-many or one-to-one setting.

In our example, `Memberships` class has two foreign keys mapping to `members.id` and `properties.id`. `Properties` class has a one-to-one relationship with `Memberships` class, so we have only one membership record per property. This is achieved with  `uselist=False` option in `memership_info` relationship. This relationship back populates the `for_property` relationship deinfed in `Memberships` class. Similarly, `associated_properties` in `Members` class and `primary_member` in `Memberships` class work in tandem.

There is a special case in the `Members` class, where there is a self-referential mapping between `primary_member_id` and `id` defined as foreign key. This lets us model hierarchical data where we have all members of a household mapped to a primary member of that household. There is a one-to-many relationship `associated_members` that will let us define all associated members to a primary member. 

<h3>Database Operations</h3>

To start performing database operations, we will need to create a session like below:

```python
from sqlalchemy.orm import sessionmaker
Session = sessionmaker(engine)
session = Session()
```

We can use this session object to then do CRUD operations on these tables. We will also define a few utility functions to view the contents of each table.

```python
def print_members():
    print("****MEMBERS")
    for instance in session.query(Members):
        print(instance.id, instance.member_name, instance.mobile_number, instance.primary_member_id)

def print_properties():
    print("****PROPERTIES")
    for instance in session.query(Properties):
        print(instance.id, instance.property_id, instance.property_type, instance.property_desc)

def print_memberships():
    print("****MEMBERSHIPS")
    for instance in session.query(Memberships):
        print(instance.id, instance.pid, instance.primary_member_id, instance.membership_date)
```

Now let us populate the tables with an example consisting of two households with three members each but with three properties, with one household owning two of these properties and remaining one owned by the other household. 

```python
m1_2=Members(member_name="Jane Doe", mobile_number="xxxx")
m1_3=Members(member_name="Nobody", mobile_number="zzzz")
m1=Members(member_name="John Doe", mobile_number="yyyy", 
            associated_members=[m1_2 ,m1_3])

m2_2=Members(member_name="John Doe2", mobile_number="xxxx2")
m2_3=Members(member_name="Nobody2", mobile_number="zzzz2")
m2=Members(member_name="Jane Doe2", mobile_number="yyyy2", 
            associated_members=[m2_2 ,m2_3])

p1=Properties(property_id="prop1",property_type="Flat",property_desc="1BHK")
p2=Properties(property_id="prop2",property_type="Flat",property_desc="2BHK")
p3=Properties(property_id="prop3",property_type="Commercial",property_desc="1000 sft unit")

mem1=Memberships(for_property=p1,primary_member=m1,membership_date=datetime.date(2003,12,1))
mem2=Memberships(for_property=p2,primary_member=m2,membership_date=datetime.date(2012,11,15))
mem3=Memberships(for_property=p3,primary_member=m2,membership_date=datetime.date(2013,2,6))

session.add_all((mem1,mem2,mem3))
session.commit()

print_members()
print_properties()
print_memberships()
```

This will output the following:

```text
****MEMBERS
1 John Doe yyyy None
2 Jane Doe2 yyyy2 None
3 Jane Doe xxxx 1
4 Nobody zzzz 1
5 John Doe2 xxxx2 2
6 Nobody2 zzzz2 2
****PROPERTIES
1 prop1 Flat 1BHK
2 prop3 Commercial 1000 sft unit
3 prop2 Flat 2BHK
****MEMBERSHIPS
1 1 1 2003-12-01 00:00:00
2 3 2 2012-11-15 00:00:00
3 2 2 2013-02-06 00:00:00
```

In the `session.add_all` command, we only needed to include the membership objects and all the parent entity records for members and properties automatically got inserted in the database along with membership records. This is indeed quite nice.

<h3>Delete Operations with CASCADE Options</h3>

If you notice all the foreign keys have `ondelete="CASCADE"` option defined, and the corresponding relationships defined on the class with the mapping primary key with options `cascade="all, delete, delete-orphan", passive_deletes=False`. As a result of this, if there is a change in primary member of a property, it will result in creation of a new membership record, and deletion of the old one, as illustrated below where we change the primary member of property `p2` from `m2` to `m1`:

```python
mem2_1=Memberships(for_property=p2,primary_member=m1,membership_date=datetime.date(2013,1,15))
session.add(mem2_1)
session.commit()

print_memberships()
```

This will output the following:

```text
****MEMBERSHIPS
1 1 1 2003-12-01 00:00:00
3 2 2 2013-02-06 00:00:00
4 3 1 2013-01-15 00:00:00
```

As you can see record 2 of memberships table is not available anymore and a new record 4  is created mapping property `pid=3` with `primary_member_id=1`. In this example, we have enforced that the Membership data only contains the latest member information.

Similarly with the removal of a primary member, the cascade options will ensure that all associated members in the `members` table and associated properites in `memberships` are deleted, as illustrated below:


```python
mem=session.get(Members, 1)
session.delete(mem)
session.commit()
print_members()
print_memberships()
```

This will output the following:

```text
****MEMBERS
2 Jane Doe2 yyyy2 None
5 John Doe2 xxxx2 2
6 Nobody2 zzzz2 2
****MEMBERSHIPS
3 2 2 2013-02-06 00:00:00
```

As you can see all the records associated with `primary_member_id=1` is deleted in `members` and `memberships` tables.

<h3>Conclusion</h3>

SQLAlchemy provides a nice and easy way to manipulate database objects through Object-Relational Mapping, and also provides nice constructs for defining entity relationships, and flexible ways of creating/updating/deleting database records in a consistent manner. 

So, this concludes the blog post. I hope you found this useful. Do drop me a note, if you have any comments/suggestions.

