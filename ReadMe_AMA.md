## LDAP Labo

## Students

- Julien Amacher
- Pierre-Alain Curty

## Structure

All object related to a person are placed in the top-level OU named **People**
We chose the following second-level OU's : **Students**, **Professors**, **Assistants** and **Admins**
In the **Students**, **Professors** and **Assistants** OU's, there is a OU for every OU discovered while parsing in the CSV file.
For instance : **TIN**, **COMEM**, **TIC**, **DEPG**, **EC** and **HEG**.

Here's our structure illustrated once imported :


![alt tag](schema.png)

Every student, professor, assistant and admin is in fact a person. In LDAP, this entity is named inetOrgPerson. The problem is, we have to store more fields. For instance : Sex, Division (TIC, HEG, etc.) and Role (Professor, Student etc.). We will then create a custom object, using the customs attributes named HEIGSex, HEIGDivision and HEIGRole, respectively. This object will be called HEIGPerson and will inheritinetOrgPerson.

![alt tag](inherit.png)

Here's the definition of our custom attributes :

**File : attrib.ldif**<br />
<code>
dn: cn=schema<br />
changetype: modify<br />
add: attributeTypes<br />
attributeTypes: ( HEIGSex-oid <br />
&nbsp;&nbsp;&nbsp;&nbsp;NAME 'HEIGSex' <br />
&nbsp;&nbsp;&nbsp;&nbsp;EQUALITY caseIgnoreMatch <br />
&nbsp;&nbsp;&nbsp;&nbsp;ORDERING caseIgnoreOrderingMatch <br />
&nbsp;&nbsp;&nbsp;&nbsp;SUBSTR caseIgnoreSubstringsMatch <br />
&nbsp;&nbsp;&nbsp;&nbsp;SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SINGLE-VALUE<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;USAGE userApplications<br />
 )<br />
attributeTypes: ( HEIGDivision-oid <br />
&nbsp;&nbsp;&nbsp;&nbsp;NAME 'HEIGDivision' <br />
&nbsp;&nbsp;&nbsp;&nbsp;EQUALITY caseIgnoreMatch <br />
&nbsp;&nbsp;&nbsp;&nbsp;ORDERING caseIgnoreOrderingMatch <br />
&nbsp;&nbsp;&nbsp;&nbsp;SUBSTR caseIgnoreSubstringsMatch <br />
&nbsp;&nbsp;&nbsp;&nbsp;SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SINGLE-VALUE<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;USAGE userApplications<br />
 )<br />
attributeTypes: ( HEIGRole-oid <br />
&nbsp;&nbsp;&nbsp;&nbsp;NAME 'HEIGRole' <br />
&nbsp;&nbsp;&nbsp;&nbsp;EQUALITY caseIgnoreMatch <br />
&nbsp;&nbsp;&nbsp;&nbsp;ORDERING caseIgnoreOrderingMatch <br />
&nbsp;&nbsp;&nbsp;&nbsp;SUBSTR caseIgnoreSubstringsMatch <br />
&nbsp;&nbsp;&nbsp;&nbsp;SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SINGLE-VALUE<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;USAGE userApplications<br />
 )<br />
</code>

Here's the definition of our custom object HEIGPerson :

**File : obj.ldif**<br />
<code>
dn: cn=schema
changetype: modify<br />
add: objectClasses<br />
objectClasses: ( HEIGPerson-oid <br />
&nbsp;&nbsp;&nbsp;&nbsp;NAME 'HEIGPerson' <br />
&nbsp;&nbsp;&nbsp;&nbsp;SUP inetOrgPerson <br />
&nbsp;&nbsp;&nbsp;&nbsp;STRUCTURAL <br />
&nbsp;&nbsp;&nbsp;&nbsp;MUST ( HEIGDivision $ HEIGRole $ HEIGSex ) <br />
 )
</code>

## How to import

1. First, generate a dataset of users using the LdapDataGeneration project.
This will yield a CSV containing all generated users.<br />Note that the project needs modification because the generated email addresses may contain some accents. This is not acceptable for the email field of a inetOrgPerson.<br /><br />To correct this, the following function has been added : <br /><code>
public String deAccent(String str) {<br />
&nbsp;&nbsp;&nbsp;&nbsp;String nfdNormalizedString = Normalizer.normalize(str, Normalizer.Form.NFD);<br />
&nbsp;&nbsp;&nbsp;&nbsp;Pattern pattern = Pattern.compile("\\p{InCombiningDiacriticalMarks}+");<br />
&nbsp;&nbsp;&nbsp;&nbsp;return pattern.matcher(nfdNormalizedString).replaceAll("");<br />
}<br />
</code><br />Source : http://stackoverflow.com/questions/1008802/converting-symbols-accent-letters-to-english-alphabet

2. Copy the CSV file in the root folder of the LDIFConverter project and run the project. You should not have a file called ou.ldif and another named users.ldif. The first one contains all organizational units that need to be created and the second contains the list of the users.<br /><br />
Example of a converted person :<br />
<code>
dn: uid=EID_100001,ou=HEG,ou=Students,ou=People,dc=example,dc=com<br />
changetype: add<br />
objectClass: HEIGPerson<br />
objectClass: top<br />
uid: EID_100001<br />
givenName: Simpson<br />
sn: Fabienne<br />
uid: EID_100001<br />
cn: Fabienne Simpson<br />
HEIGSex: FEMALE<br />
mail: fabienne.simpson@heig-vd.ch<br />
telephoneNumber: (024) 777 593 383<br />
HEIGDivision: HEG<br />
HEIGRole: Student<br />
</code>

3. Then, create a new backend on the LDAP server uisng this command : <br /><br /><code>dsconfig.bat create-backend --set base-dn:dc=example,dc=com --set enabled:true --type local-db --backend-name myBackend --hostname 127.0.0.1 --port 4444 --bindDN "cn=Directory Manager" --bindPassword 1234 -X -n</code>

4. Import the custom attributes, the custom HEIGPerson object, the OU's and all users using the following commands, in order : <br /><br /><code>ldapmodify.bat --port 389 --bindDN "cn=Directory Manager" --bindPassword 1234 --filename attrib.ldif<br />
ldapmodify.bat --port 389 --bindDN "cn=Directory Manager" --bindPassword 1234 --filename obj.ldif<br />
ldapmodify.bat --port 389 --bindDN "cn=Directory Manager" --bindPassword 1234 --filename ou.ldif<br />
ldapmodify.bat --port 389 --bindDN "cn=Directory Manager" --bindPassword 1234 --filename users.ldif</code>


## Questions

**What is the number (not the list!) of people stored in the directory?**
<code>
ldapsearch --hostname 127.0.0.1 --port 389 --bindDN "cn=Directory Manager" --bindPassword 1234 --countentries --baseDN "dc=example,dc=com" "(objectclass=HEIGPerson)" ldapentrycount
</code>

Result: 10'000


**What is the number of departments stored in the directory?**
<code>
ldapsearch --hostname 127.0.0.1 --port 389 --bindDN "cn=Directory Manager" --bindPassword 1234 --countentries --baseDN "ou=Students,dc=example,dc=com" "(objectclass=organizationalUnit)" ldapentrycount
</code>

Result: 23

**What is the list of people who belong to the TIC Department?**
<code>ldapsearch --hostname 127.0.0.1 --port 389 --bindDN "cn=Directory Manager" --bindPassword 1234 --baseDN "dc=example,dc=com" "(HEIGDivision=TIC)"
</code>

**What is the list of students in the directory?**
<code>ldapsearch --hostname 127.0.0.1 --port 389 --bindDN "cn=Directory Manager" --bindPassword 1234 --baseDN "dc=example,dc=com" "(HEIGRole=Student)"</code>

**What is the list of students in the TIC Department?**
<code>ldapsearch --hostname 127.0.0.1 --port 389 --bindDN "cn=Directory Manager" --bindPassword 1234 --baseDN "dc=example,dc=com" "(&(HEIGDivision=TIC)(HEIGRole=Student))"
</code>

**What command do you run to define a dynamic group that represents all members of the TIN Department?**

**File : TINMembers.ldif**<br />
<code>
dn: cn=TINMembers,dc=example,dc=com<br />
changetype: add<br />
cn: TINmembers<br />
objectClass: top<br />
objectClass: groupOfURLs<br />
ou: People<br />
memberURL: ldap:///ou=People,dc=example,dc=com??sub?(HEIGDivision=TIN)
</code>
<br />
<br />
<code>
ldapmodify --port 389 --bindDN "cn=Directory Manager" --bindPassword 1234 --defaultAdd --filename TINMembers.ldif
</code>

**What command do you run to get the list of all members of the TIN Department?**

<code>
ldapsearch --port 389 --bindDN "cn=Directory Manager" --bindPassword 1234 --baseDN dc=example,dc=com "(&(objectclass=HEIGPerson)(isMemberOf=cn=TINMembers,ou=People,dc=example,dc=com))"
</code>

**What command do you run to define a dynamic group that represents all students with a last name starting with the letter 'A'?**

**File : StudentsLastNameStartingWithA.ldif**<br />
<code>
dn: cn=StudentsLastNameStartingWithA,dc=example,dc=com<br />
changetype: add<br />
cn: StudentsLastNameStartingWithA<br />
objectClass: top<br />
objectClass: groupOfURLs<br />
ou: People<br />
memberURL: ldap:///ou=People,dc=example,dc=com??sub?(&(HEIGRole=Student)(givenName=A*))
</code>
<br />
<br />
<code>
ldapmodify --port 389 --bindDN "cn=Directory Manager" --bindPassword 1234 --defaultAdd --filename StudentsLastNameStartingWithA.ldif
</code>

**What command do you run to get the list of these students?**

<code>
ldapsearch --port 389 --bindDN "cn=Directory Manager" --bindPassword 1234 --baseDN dc=example,dc=com "(&(objectclass=HEIGPerson(isMemberOf=cn=StudentsLastNameStartingWithA,ou=People,dc=example,dc=com))"
</code>
