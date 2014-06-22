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

Here an image to show our personal object:

<center><img width=520 src="ObjectsPers/.png"></center>


# Data generation and import

To generate the data we have used the NetBean project LdapDataGeneration. If we run the main program we will generate a .csv file that contains all HEIG-VD person that we want to manage trough our LDAP system.

Once the .csv file is generated we need a simple system that will convert all entry of the file in valid LDAP entry. To achieve this step we have created a new NetBean project named : LdapParsing. 

The main goal of this program is to generate .ldif files that can be imported using openDJ in our LDAP server.

The first file generated is "structure.ldif" and can be used to import the organizational units needed.

The second generated file is "userLdif" and contains all HeigVDperson that are finded in the .csv file.

To avoid problems generated by special characters (à,è,ö,ì ...) in email entry we have added a function on our code that allow to normalize the string.

Now that we have generated the needed .ldif we can import them using the GUI buttons on OpenDJ.

```
We have to import:
	-customs attributes
	-custom objects
	-structural data 
	-users data
```

# Questions

**What is the number (not the list!) of people stored in the directory?**
<code>
ldapsearch --hostname 127.0.0.1 --port 389 --bindDN "cn=Directory Manager" --bindPassword toor --countentries --baseDN "dc=heigvd,dc=.ch" "(objectclass=HeigVDperson)" ldapentrycount
</code>

Result: 10'000


**What is the number of departments stored in the directory?**
<code>
ldapsearch --hostname 127.0.0.1 --port 389 --bindDN "cn=Directory Manager" --bindPassword toor --countentries --baseDN "ou=Departments,dc=heigvd,dc=.ch" "(objectclass=organizationalUnit)" ldapentrycount
</code>

Result: 6

**What is the list of people who belong to the TIC Department?**
<code>ldapsearch --hostname 127.0.0.1 --port 389 --bindDN "cn=Directory Manager" --bindPassword toor --baseDN "dc=heigvd,dc=.ch" "(HeigVDdepartment=TIC)"
</code>

**What is the list of students in the directory?**
<code>ldapsearch --hostname 127.0.0.1 --port 389 --bindDN "cn=Directory Manager" --bindPassword toor --baseDN "dc=heigvd,dc=.ch" "(HeigVDrole=Student)"</code>

**What is the list of students in the TIC Department?**
<code>ldapsearch --hostname 127.0.0.1 --port 389 --bindDN "cn=Directory Manager" --bindPassword toor --baseDN "dc=heigvd,dc=.ch" "(&(HeigVDdepartment=TIC)(HeigVDrole=Student))"
</code>

**What command do you run to define a dynamic group that represents all members of the TIN Department?**

**File : TINMembers.ldif**<br />
<code>
dn: cn=TINMembers,dc=heigvd,dc=.ch<br />
changetype: add<br />
cn: TINmembers<br />
objectClass: top<br />
objectClass: groupOfURLs<br />
ou: People<br />
memberURL: ldap:///ou=People,dc=heigvd,dc=.ch??sub?(HeigVDdepartment=TIN)
</code>
<br />
<br />
<code>
ldapmodify --port 389 --bindDN "cn=Directory Manager" --bindPassword toor --defaultAdd --filename TINMembers.ldif
</code>

**What command do you run to get the list of all members of the TIN Department?**

<code>
ldapsearch --port 389 --bindDN "cn=Directory Manager" --bindPassword toor --baseDN dc=heigvd,dc=.ch "(&(objectclass=HEIGPerson)(isMemberOf=cn=TINMembers,ou=People,dc=heigvd,dc=.ch))"
</code>

**What command do you run to define a dynamic group that represents all students with a last name starting with the letter 'A'?**

**File : StudentsLastNameStartingWithA.ldif**<br />
<code>
dn: cn=StudentsLastNameStartingWithA,dc=heigvd,dc=.ch<br />
changetype: add<br />
cn: StudentsLastNameStartingWithA<br />
objectClass: top<br />
objectClass: groupOfURLs<br />
ou: People<br />
memberURL: ldap:///ou=People,dc=heigvd,dc=.ch??sub?(&(HeigVDrole=Student)(givenName=A*))
</code>
<br />
<br />
<code>
ldapmodify --port 389 --bindDN "cn=Directory Manager" --bindPassword toor --defaultAdd --filename StudentsLastNameStartingWithA.ldif
</code>

**What command do you run to get the list of these students?**

<code>
ldapsearch --port 389 --bindDN "cn=Directory Manager" --bindPassword toor --baseDN dc=heigvd,dc=.ch "(&(objectclass=HEIGPerson(isMemberOf=cn=StudentsLastNameStartingWithA,ou=People,dc=heigvd,dc=.ch))"
</code>