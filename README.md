# Lab 9: LDAP

- Righitto Simone
- Roubaty Anthony

# Structure

The first thing that we have done after having generated the .csv file is to define a structure for our LDAP:

Basically we have chosen a flat structure that include:



```
As top-level an OU named People.
Then on the second level we have defined: 
	- Students
	- Professors
	- Assistants
	- Admins

And finally on every second level entry we have the followings OU:
	- TIN
	- TIC
	- COMEM
	- EC
	- HEG
	- DEPG
```

The main problem with our organization is that default inetOrgPerson class doesn't contains all the needed attributes (sex, role and department). So we have to create custom objects that will contains our custom attributes. So we have defined the followings custom attributes:
```
	- HeigVDsex
	- HeigVDdepartment
	- HeigVDrole
```
And we have added those custom attributes to our custom object HeigVDperson.


# Data generation and import

To generate the data we have used the NetBean project LdapDataGeneration. If we run the main program we will generate a .csv file that contains all HEIG-VD person that we want to manage trough our LDAP system.

Once the .csv file is generated we need a simple system that will convert all entry of the file in valid LDAP entry. To achieve this step we have created a new NetBean project named : LdapParsing. 

The main goal of this program is to generate .ldif files that can be imported using openDJ in our LDAP server.

