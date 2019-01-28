###################################
OEP-32: Unique Identifier for Users
###################################

.. list-table::
   :widths: 25 75

   * - OEP
     - :doc:`OEP-32 <oep-0032-arch-unique-identifier-for-users>`
   * - Title
     - Unique Identifier for Users
   * - Last Modified
     - 2019-01-28
   * - Authors
     - Brian Wilson, Robert Raposa, Nimisha Asthagiri
   * - Arbiter
     - Olga Stroilova
   * - Status
     - Draft
   * - Type
     - Architecture
   * - Created
     - 2019-01-25
   * - `Review Period`
     - 2019-01-28 - 2019-02-11

Decision
========

We will use the Open edX user_id (a.k.a. LMS user_id) to uniquely identify a user in the Open edX system. The Open edX user_id is the **id** value of the user's row in the `Django auth_user`_ table in the Open edX LMS.

The LMS user_id should be used for all internal and public events, REST APIs, and JavaScript APIs.

.. _Django auth_user: https://docs.djangoproject.com/en/2.0/topics/auth/default/#user-objects


Context
=======

In 2016, the analytics team wrote a decision to use the the LMS user_id in `Using consistent user identifiers in Segment events`_.

In addition to needing a consistent user identifier for internal events, we also need one for public events, REST APIs and JavaScript APIs.

.. _Using consistent user identifiers in Segment events: https://openedx.atlassian.net/wiki/spaces/AN/pages/144441849/Using+consistent+user+identifiers+in+Segment+events


User Identifier Requirements
----------------------------

The requirements for a User Identifier as described by the analytics decision document, are also applicable for our REST APIs and JavaScript APIs.

* Uniqueness: the identifier must not represent more than one user.

* The identifier must map back to other information about the user (e.g. demographics, enrollment, etc.).

* The identifier for a given user should not change.

* The identifier should not contain `Personally Identifiable Information (PII)`_.

* The identifier should be consistent across all systems, including the LMS and Studio, mobile apps for iOS and Android, and other Open edX Services like Ecommerce.

.. _Personally Identifiable Information (PII): oep-0030-arch-pii-markup-and-auditing.rst


User Identifier Candidate Comparison
------------------------------------

The following table summarizes the comparisons of a list of candidate user identifiers.

.. list-table::
   :header-rows: 1
   :widths: 25 75

   * - Identifier
     - Limitations
   * - username
     - Username is considered PII.
   * - LMS user_id
     - Information leakage of auth_user table with easily guessable values; tied down to implementation of the housing database.
   * - email address
     - Is PII and modifiable by the user.
   * - anonymous user_id
     - It is currently constructed by hashing the user's LMS user_id with the Django server's *SECRET_KEY* value. This value will change when the *SECRET_KEY* is rotated.

       *Note*: Although this id was named the "anonymous user_id", it is no more anonymous than the "LMS user_id" from a PII perspective.

As noted above, the LMS user_id was selected for analytics as the best match given the requirements.

Drawbacks of the LMS user_id
----------------------------

As noted in the `User Identifier Candidate Comparison`_ above, using the LMS user_id has the following two drawbacks:

* Information leakage of auth_user table with easily guessable values.

* The id is tied down to implementation of the housing database.

Cost of an alternative to the LMS user_id
-----------------------------------------

If we were to try to eliminate the drawbacks of using the auto-incremented LMS user_id as our unique identifier, an alternative might be to introduce a new id, like a UUID (Universally Unique IDentifier), that did not have the `Drawbacks of the LMS user_id`_.

However, the introduction of a new id has large costs:

* Introducing a brand new id as the standard user identifier would mean starting from scratch regarding compliance. The effort to reach even the current level of compliance with use of the LMS user_id throughout the Open edX system would be large.

* Data analytics is one area where, even though we don't have 100% compliance with use of the LMS user_id, we have enough compliance to make the data useful. Switching the user identifier for data analytics would require a large coordinated effort that would be difficult for data scientists with no benefit to them.

* Adding another user id without being able to retire an old id also has the drawback of making the system less approachable, as each developer tries to learn which id to use in which situation.

Note: A plan for reaching compliance with any given unique user id, in all cases, is out of scope of this OEP.

Consequences
============

The decision to continue to use the LMS user_id as the unique user id was determined because the cost of introducing a new id far outweighed the existing drawbacks of using the LMS user_id from a practical standpoint.

As part of this decision, the `LMS OAuth Dispatch App`_ will start including the LMS user_id in the JWTs (JSON Web Tokens) used as part of the `Open edX Authentication`_.

.. _LMS OAuth Dispatch App: https://github.com/edx/edx-platform/blob/master/openedx/core/djangoapps/oauth_dispatch/docs/README.rst
.. _Open edX Authentication: https://openedx.atlassian.net/wiki/spaces/PLAT/pages/160912480/Open+edX+Authentication

Pros and Cons
-------------

Pros:
~~~~~

* Eliminates the need for a large transition effort to a new id.

* Simplifies the codebase, avoiding trying to understand when to use an old id vs a new id.

Cons:
~~~~~

Leaves the following drawbacks of the LMS user_id in place:

* Information leakage of auth_user table with easily guessable values.

* The id is tied down to implementation of the housing database.

Clarifications
--------------

The LMS user_id can be used in any case that one might need an "anonymous" user id, because the LMS user_id is not considered PII. In general, without any additional requirements, the LMS user_id should be the unique user id for any integrations with a third-party system. This OEP states that the actual drawbacks of the LMS user_id, information leakage and being tied down to the implementation of the database, are known issues with this id, and can be disregarded even in third-party integrations.

There are legacy user ids in the Open edX Platform that use the term "anonymous" in the name, but they are no more or less anonymous from a PII perspective.

One example of an alternative user id, called an "anonymous id", is used in the legacy implementation of LTI. From a PII perspective, the LTI anonymous id is no more anonymous than the LMS user_id. However, the current LTI implementation, which uses an id made of a combination of user id and course id, makes it impossible for a third party system to build a user model in its own system across courses. This also clearly limits the capabilities a third-party system might offer a user. It is out of scope of this OEP to state whether or not this is the right choice for LTI or for any other third-party integration. However, a different id should only be considered in a use case with additional requirements that are not addressed in this OEP.

This OEP clarifies that the LMS user_id is considered a safe option for third-party integrations as described above. One example of a third-party integration where the LMS user_id can now safely be used is in the `Realtime Events API`_.

.. _Realtime Events API: oep-0026-arch-realtime-events.rst
