**Projet LDAP - Installation et Configuration (Jusqu'au Job 2)**

## **1. Introduction**
Voici les étapes d'installation et de configuration d'un serveur LDAP pour le domaine `datatrust.local`. Il inclut la mise en place du serveur, la création des unités organisationnelles, ainsi que l'ajout des utilisateurs et des groupes (admin, dev et user).

---

## **2. Installation du Serveur LDAP**

### **1️ Installation des paquets nécessaires**
Sur un serveur Debian/Ubuntu, installez OpenLDAP et les outils associés :
```bash
apt update && apt install slapd ldap-utils -y
```

### **2️ Configuration du serveur LDAP**

Une fois l'installation terminée, lancez la configuration interactive :
```bash
dpkg-reconfigure slapd
```
- **Nom de domaine DNS** : `datatrust.local`
- **Nom de l'organisation** : `DataTrust`
- **Mot de passe administrateur LDAP** : [Définir un mot de passe sécurisé]
- **Type de base de données** : MDB
- **Supprimer la base de données lors de la suppression de slapd** : Non
- **Autoriser le protocole LDAPv2** : Non

Après la configuration, redémarrez le service :
```bash
systemctl restart slapd
```

---

## **3. Création de la Structure LDAP**

### **1️ Création des Unités Organisationnelles (OU)**
Créez un fichier `base.ldif` contenant la structure de base :
```ldif
dn: ou=users,dc=datatrust,dc=local
objectClass: organizationalUnit
ou: users

dn: ou=developers,dc=datatrust,dc=local
objectClass: organizationalUnit
ou: developers

dn: ou=admins,dc=datatrust,dc=local
objectClass: organizationalUnit
ou: admins

dn: ou=groups,dc=datatrust,dc=local
objectClass: organizationalUnit
ou: groups
```
Ajoutez cette structure avec :
```bash
ldapadd -x -D "cn=admin,dc=datatrust,dc=local" -W -f base.ldif
```

---

## **4. Création des Groupes LDAP**

Créez un fichier `groups.ldif` pour ajouter les groupes `admin`, `dev` et `user` :
```ldif
dn: cn=Users,ou=groups,dc=datatrust,dc=local
objectClass: top
objectClass: posixGroup
cn: Users
gidNumber: 5001

dn: cn=Developers,ou=groups,dc=datatrust,dc=local
objectClass: top
objectClass: posixGroup
cn: Developers
gidNumber: 5002

dn: cn=Administrators,ou=groups,dc=datatrust,dc=local
objectClass: top
objectClass: posixGroup
cn: Administrators
gidNumber: 5003
```
Ajoutez ces groupes avec :
```bash
ldapadd -x -D "cn=admin,dc=datatrust,dc=local" -W -f groups.ldif
```

---

## **5. Création des Utilisateurs**

Créez un fichier `users.ldif` pour ajouter un administrateur, un développeur et un utilisateur basique :
```ldif
dn: uid=alice,ou=users,dc=datatrust,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: top
cn: Alice
sn: Alice
uid: alice
uidNumber: 1000
gidNumber: 5001
homeDirectory: /home/alice
userPassword: {SSHA}tNqrD3LwnHV99k9DvuaIwhD30UJxRo9G

dn: uid=bob,ou=developers,dc=datatrust,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: top
cn: Bob
sn: Bob
uid: bob
uidNumber: 1001
gidNumber: 5002
homeDirectory: /home/bob
userPassword: {SSHA}QF7YEmh1DST9TRRD52dkdhLfCOQJTWNe

dn: uid=claire,ou=admins,dc=datatrust,dc=local
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: top
cn: Claire
sn: Claire
uid: claire
uidNumber: 1002
gidNumber: 5003
homeDirectory: /home/claire
userPassword: {SSHA}Umnr/q+QpLRva03XEPDZaA7+EK5CivrJ
```
Ajoutez ces utilisateurs avec :
```bash
ldapadd -x -D "cn=admin,dc=datatrust,dc=local" -W -f users.ldif
```

---

## **6. Vérification des Entrées LDAP**

### **Lister tous les utilisateurs**
```bash
ldapsearch -x -LLL -b "ou=users,dc=datatrust,dc=local" uid cn sn
```

### **Lister les groupes et leurs membres**
```bash
ldapsearch -x -LLL -b "ou=groups,dc=datatrust,dc=local" cn uniqueMember
```

### **Tester l'authentification d'un utilisateur**
```bash
ldapwhoami -x -D "uid=alice,ou=users,dc=datatrust,dc=local" -W
```
Si tout est bien configuré, la sortie doit être :
```
ud=alice,ou=users,dc=datatrust,dc=local
```

---

## **7. Conclusion**
À ce stade, nous avons :
Installé et configuré OpenLDAP pour `datatrust.local`.
Créé les structures LDAP (OU, Groupes, Utilisateurs).
Vérifié les entrées LDAP et testé l'authentification.

 **Prochaine étape (Job 3) : Mise en place des options de sécurité LDAP (ppolicy, verrouillage des comptes, expiration des mots de passe).**

