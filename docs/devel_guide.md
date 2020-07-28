# Developer's Guide

The Security Layer relies on FIWARE components that are all Open Source, fully documented with an existing community managing them. It’s always possible (as in any open source project) to propose evolution and corrections.

* Keyrock : [https://github.com/ging/fiware-idm](https://github.com/ging/fiware-idm)
* Wilma : [https://github.com/ging/fiware-pep-proxy](https://github.com/ging/fiware-pep-proxy)
* AuthZForce : [https://github.com/authzforce/core](https://github.com/authzforce/core)

## Potential extensions
The current version of KeyRock uses a MySQL Database to manage the objects. An evolution could be to propose a different driver database to allow KeyRock to work with an external IdM solution like LDAP to be more closely integrated in the port IT solution.


## Additional notes
FIWARE propose documentation to easily integrate their solutions: 
* KeyRock documentation guide : [https://fiware-idm.readthedocs.io/en/latest/index.html](https://fiware-idm.readthedocs.io/en/latest/index.html)
* Tutorials : [https://github.com/FIWARE/tutorials.Identity-Management](https://github.com/FIWARE/tutorials.Identity-Management)
