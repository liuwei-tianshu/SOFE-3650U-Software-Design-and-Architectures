# SLocator: Localizing the Origin of SQL Queries in Database-Backed Web Applications
> **Note:** This is the source code (**SLocator**), demo (**Demo**), experiment (**Experiment**) and evaluation dataset (**Dataset**) for SLocator
> 
> **Note:** The **appendix.pdf** is in (**Appendix**)

## 1. SLocator
This directory constians **Statically Inferring Database Access**, the static analysis to infer the database access of each control flow path in the source code.

We use [Crystal](https://code.google.com/archive/p/crystalsaf/), a Java static analysis framework that is built on top of Eclipse JDT, to analyze the source code and extract the CFG.

## 2. Demo
This directory contains **Locating the Paths that Generate a Given SQL Query**. Given SQL queries, we use information retrieval techniques to rank the control flow paths that have the highest database access similarity.

We give a demo to show how SLocator works on PetClinic.

#### 2.1 Requirement
- Python 3.7
- Dependency: sklearn nltk sqlparse

#### 2.2 Install and Running
Download whole directory **Demo**.

Options:
```
  -s, --sql <SQL query>  Set the SQL query to be located. 
                         1 SQL session log (should be separated with |)
                         2 individual query log
```

**Command for SQL session log**: python SLocator.py -s "select distinct owner0_.id as id1_0_0_, pets1_.id as id1_1_1_, owner0_.first_name as first_na2_0_0_, owner0_.last_name as last_nam3_0_0_, owner0_.address as address4_0_0_, owner0_.city as city5_0_0_, owner0_.telephone as telephon6_0_0_, pets1_.name as name2_1_1_, pets1_.birth_date as birth_da3_1_1_, pets1_.owner_id as owner_id4_1_1_, pets1_.type_id as type_id5_1_1_, pets1_.owner_id as owner_id4_0_0__, pets1_.id as id1_1_0__ from owners owner0_ left outer join pets pets1_ on owner0_.id=pets1_.owner_id where owner0_.last_name like 'xxxx'|select pettype0_.id as id1_3_0_, pettype0_.name as name2_3_0_ from types pettype0_ where pettype0_.id=1|select visits0_.pet_id as pet_id4_1_0_, visits0_.id as id1_6_0_, visits0_.id as id1_6_1_, visits0_.visit_date as visit_da2_6_1_, visits0_.description as descript3_6_1_, visits0_.pet_id as pet_id4_6_1_ from visits visits0_ where visits0_.pet_id=1"

The output of SLocator is as following (ranked top 5 paths):
```
Given SQL queries:
select distinct owner0_.id as id1_0_0_, pets1_.id as id1_1_1_, owner0_.first_name as first_na2_0_0_, owner0_.last_name as last_nam3_0_0_, owner0_.address as address4_0_0_, owner0_.city as city5_0_0_, owner0_.telephone as telephon6_0_0_, pets1_.name as name2_1_1_, pets1_.birth_date as birth_da3_1_1_, pets1_.owner_id as owner_id4_1_1_, pets1_.type_id as type_id5_1_1_, pets1_.owner_id as owner_id4_0_0__, pets1_.id as id1_1_0__ from owners owner0_ left outer join pets pets1_ on owner0_.id=pets1_.owner_id where owner0_.last_name like 'xxxx'
select pettype0_.id as id1_3_0_, pettype0_.name as name2_3_0_ from types pettype0_ where pettype0_.id=1
select visits0_.pet_id as pet_id4_1_0_, visits0_.id as id1_6_0_, visits0_.id as id1_6_1_, visits0_.visit_date as visit_da2_6_1_, visits0_.description as descript3_6_1_, visits0_.pet_id as pet_id4_6_1_ from visits visits0_ where visits0_.pet_id=1

Pre-processed SQL queries:
select distinct from owners left outer join pets where owner.last_name like ? 
select from types where pettype.id=? 
select from visits where visits.pet_id=?

-------Top1 ranked control flow path according to similarity score (Only top 5 are presented here) ------
request:org.springframework.samples.petclinic.web.ownercontroller.processfindform(owner,bindingresult,map)
method:org.springframework.samples.petclinic.service.clinicserviceimpl.findownerbylastname(string)
method:org.springframework.samples.petclinic.repository.jpa.jpaownerrepositoryimpl.findbylastname(string)
[select distinct owner from owner owner left join fetch owner.pets where owner.lastname like :lastname]
[select  from types where id=?]
[select visit_date, description, null from visits where id=?]
Syntactic Similarity:0.6198747639278099
Semantic Similarity:1.0
Combining Similarity Score (Syntactic Similarity + Semantic Similarity): 1.61987476392781

--------Top2 ranked control flow path according to similarity score (Only top 5 are presented here) -------
...
```

**Command for individual query log**: python SLocator.py -s "select owner0_.id as id1_0_1_, owner0_.first_name as first_na2_0_1_, owner0_.last_name as last_nam3_0_1_, owner0_.address as address4_0_1_, owner0_.city as city5_0_1_, owner0_.telephone as telephon6_0_1_, pets1_.owner_id as owner_id4_0_3_, pets1_.id as id1_1_3_, pets1_.id as id1_1_0_, pets1_.name as name2_1_0_, pets1_.birth_date as birth_da3_1_0_, pets1_.owner_id as owner_id4_1_0_, pets1_.type_id as type_id5_1_0_ from owners owner0_ left outer join pets pets1_ on owner0_.id=pets1_.owner_id where owner0_.id=1" 

The output of SLocator is as following (ranked top 5 paths):
```
Given SQL queries:
select owner0_.id as id1_0_1_, owner0_.first_name as first_na2_0_1_, owner0_.last_name as last_nam3_0_1_, owner0_.address as address4_0_1_, owner0_.city as city5_0_1_, owner0_.telephone as telephon6_0_1_, pets1_.owner_id as owner_id4_0_3_, pets1_.id as id1_1_3_, pets1_.id as id1_1_0_, pets1_.name as name2_1_0_, pets1_.birth_date as birth_da3_1_0_, pets1_.owner_id as owner_id4_1_0_, pets1_.type_id as type_id5_1_0_ from owners owner0_ left outer join pets pets1_ on owner0_.id=pets1_.owner_id where owner0_.id=1

Pre-processed SQL queries:
select from owners left outer join pets where owner.id=? 

------Top1 ranked control flow path according to similarity score (Only top 5 are presented here) -----------
request:org.springframework.samples.petclinic.web.ownercontroller.showowner(int)
method:org.springframework.samples.petclinic.service.clinicserviceimpl.findownerbyid(int)
method:org.springframework.samples.petclinic.repository.jpa.jpaownerrepositoryimpl.findbyid(int)
[select owner from owner owner left join fetch owner.pets where owner.id =:id]
[select  from types where id=?]
[select visit_date, description, null from visits where id=?]
Syntactic Similarity:0.7683219433653732
Semantic Similarity:0.3333333333333333
Combining Similarity Score (Syntactic Similarity + Semantic Similarity): 1.1016552766987064

-------Top2 ranked control flow path according to similarity score (Only top 5 are presented here) ---------
...
```

