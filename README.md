# Lab 9: LDAP

- Righitto Simone
- Roubaty Anthony

# Structure

The first thing that we have done after having generated the .csv file is to define a structure for our LDAP:

We have chosen the following structure:

```
	 HEIG-VD
	    |
	  /   \
	 /     \
  People   Departments
```



```
As top-level we have defined DC=heigvd, DC=.ch
Then on the second level we have defined the followings OU: 
	- People
	- Departments

```

Now we have to choose the needed attributes for ours OU.

```
For People we have:
	-id
	-nom
	-prenom
	-phone
	-email
	-sex
	-department
	-role
	
And for Departments:
	-id
	-nom
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

The first file generated is "structure.ldif" and can be used to import the organizational units needed.

The second generated file is "userLdif" and contains all HeigVDperson that are finded in the .csv file.

Now that we have generated the needed .ldif we can import them using the GUI buttons on OpenDJ.

```
We have to import:
	-customs attributes
	-custom objects
	-structural data 
	-users data
```

# Questions



