Introduction
============

Originally the Persistence API (in the module ``persist/core/api``) was completely implemented
by a self-made persistence framework.

This implementation has now been replaced by Hibernate (currently version 5.2.x), due to the following reasons:

* Lack of documentation and test coverage
* Lost knowledge (original developers no longer work here)
* Missing features (complex queries, typed entities)

This document describes...

* how the existing API has been implemented using Hibernate as backend (necessary to support existing code)
* the new API

This document does not describe the usage of the old Persistence API or Hibernate itself.