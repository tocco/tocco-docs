Calendar
========

Entities
--------
The base entities of the calendar system are ``Calendar`` and ``Calendar_event``. Calendars exist in two variants,
system-wide and entity-specific.

System calendars exist only once per type and all events are recorded in that calendar. Such calendars exist for the
following types:

* **User**
    contains all the birthdates in the system
* **Issue**
    for each entity a ``Calendar_event`` from the start date to the end date is created
* **Task**
    for each entity a ``Calendar_event`` from the earliest start date to the planned end date is created
* **Timereport_scheme_day**
    this entity contains things like holidays and weekends, so for each of these a ``Calendar_event`` is created

Entity-specific calendars exist once for each entity (called its source) per type and only the events belonging to the
source are recorded in the calendar. Such calendars exist for the following types:

* **User (Participant)**
    such a calendar exists for each ``User`` that is referenced from a ``Reservation_registration``
    through ``relRegistration.relUser``, for each ``Reservation_registration`` a ``Calendar_event`` with the dates of
    its reservation is created and added to the calendar of its user
* **User (Lecturer)**
    such a calendar exists for each ``User`` that is referenced from a ``Reservation_lecturer_booking``
    through ``relLecturer_booking.relUser``, for each ``Reservation_lecturer_booking`` a ``Calendar_event`` with the
    dates of its reservation is created and added to the calendar of its user
* **Room**
    for each ``Reservation`` related to a ``Room`` a ``Calendar_event`` is created and added to the calendar of the
    ``Room``
* **Appliance**
    for each ``Reservation`` related to a ``Appliance`` a ``Calendar_event`` is created and added to the
    calendar of the ``Appliance``
* **Event**
    for each ``Reservation`` related to an ``Event`` a ``Calendar_event`` is created and added to the calendar of the
    ``Event``

All these calendar types are configured in the configuration point ``nice2.optional.calendar.CalendarEvents``.

Listeners
---------
There are three listeners that are responsible to keep all ``Calendar_event`` entities up-to-date.

* :abbr:`GenericCalendarEventListener (ch.tocco.nice2.optional.calendar.impl.entitylistener.calendarevent.GenericCalendarEventListener)`
  generates ``Calendar_event`` entities for system calendars.
* :abbr:`EntityCalendarEventListener (ch.tocco.nice2.optional.calendar.impl.entitylistener.calendarevent.EntityCalendarEventListener)`
  generates ``Calendar_event`` entities for entity-specific calendars.
* :abbr:`OfftimeEventMappingEntityListener (ch.tocco.nice2.optional.conflict.impl.OfftimeEventMappingEntityListener)`
  creates ``Calendar_event`` entities for ``Offtime_event`` entities. If the ``Offtime_event`` has no related
  ``Calendar`` the ``Calendar_event`` entities are created in every existing ``Calendar``.

In all listeners ``Calendar_events`` entities are created, updated (actually recreated) and deleted when the entities,
from which the data for the events are pulled, get adjusted.

In addition, there are four contributions to ``nice2.entityoperation.CascadingDeleteContribution`` so that
``Calendar_event`` entities are deleted when ``Registration``, ``Lecturer_booking``, ``Reservation_registration`` or
``Reservation_lecturer_booking`` are deleted.

Conflict
--------
``Conflict`` entities are related to two ``Calendar_event`` entities in the same calendar which overlap. These are
created in :abbr:`CreateConflictEntityListener (ch.tocco.nice2.optional.conflict.impl.entitylistener.CreateConflictEntityListener)`.
Since ``Calendar_event`` entities are only recreated or deleted there are no listeners that update or delete ``Conflict``
entities, they are just cascade deleted with their ``Calendar_event`` entities.

:abbr:`SetConflictsStatusListener (ch.tocco.nice2.optional.conflict.impl.entitylistener.SetConflictsStatusListener)`
sets ``Conflict_status`` on ``Registration``, ``Lecturer_booking`` and ``Reservation``, depending on the existence
of ``Conflict`` entities related to them.



