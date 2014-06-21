# Lab 9: LDAP

- Righitto Simone
- Roubaty Anthony

# Structure

The first thing that we have done after having generated the .cvs file is to define a structure for our LDAP:

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


