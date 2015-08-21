

#############################

#############################

deployment scenarios for two authentication strategies, 

.. contents::
   :local:
   :depth: 1



***************************************************
Deploying for Anonymous User Authentication
***************************************************

LTI supports anonymous use by creating and logging in dynamically generated
users. In this deplyment scenario, learners log in to the remote learning
system, and session information is shared between that system and your Open edX
implementation. Learners who access both native content and content provided
through LTI by the Open edX implementation can see the pseudo-anonymous sign in
information created for them by the system.

When you use the anonymous user authentication strategy, you can provide a
transparent authentication process that offers a simpler experience for users.
To do so, you deploy an instance of the Open edX platform only for use as an
LTI tool provider. This instance is a separate domain, or microsite, that
isolates LTI requests from regular sessions in the LMS.

In this scenario, the edX instance that is the tool provider is used solely for
asset management. Learners do not use the LMS to enroll or access courses.

If you already have a production edX instance with a repository
of courses and course content, existing course content can be reused: export
the course, import it into the LTI-only instance, and then set the course end
date to a date in the past.


The Open edX LTI Provider is designed to make user management transparent to the end user. When a user first makes an LTI launch request, edX creates a new user account with a random username and password for that LTI identity. On subsequent LTI launches, edX will associate the LTI user with the existing edX account, and log that user in. It is not expected that users will ever have to visit the edX server itself, since all interactions will be mediated over LTI. In the case of an Open edX server that handles both LTI and regular edX content this may not be true, and users may find themselves logged in with a meaningless user name. If so they will have to log out and back in again with their edX identity.



***************************************************
Deploying for Full User Authentication
***************************************************



