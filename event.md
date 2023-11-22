# Event Mgmt and Search

Events Management and Search:

----

**4.1. Event Organization**<br>

The system allows users to create, edit, and delete events (users can only edit/delete events created by themselves). The details of each event must contain its name, the date, time, location it's taking place, a set of topics (interests) it's related to, the ticket price (if any), a textual description, and the visibility type (public/private). 

----

**4.2. Event Visibility**<br>

When creating an event, the user can set it as public (visible to all the users) or private (associated with a list of usernames and will only be visible to these users).

----

**4.3. Event Attendance**<br>

Any user can mark an event that they can visualize as "not going" (option by default), "interested", or "going", and update this value for events that didn’t yet take place. 

----

**4.4. Future Events Search**<br>

The system allows users to search for nearby future events from a list. 
Note that this search aims for the user to find events to attend, so the retrieved events must be in a given radius from the user's current location.

----

**4.5. Selected Future Events Search**<br>

The system allows users to visualize future events marked as ‘interested’ or ‘going’. In this case, the proximity criteria are not applicable because users may change their location after selecting some of the events, thus potentially moving them outside the predefined 'nearby' radius. 

----

**4.6. Past Events Search**<br>

The system allows a user to visualize the list of past events marked as 'interested' or 'going'. The only interaction allowed from this list is to display the details of any event. 

----

**4.7. Created Events Search**<br>

The system allows a user to visualize the list of created events. This list may include future or past events. Indeed, past events in this list are read-only, i.e., they cannot be updated. 

----

**4.8. Event Lists Operations**<br>

From all the event lists, users can access any event and perform any authorized operation depending on whether the event is future or past, event ownership, visibility, etc. Note that removed events have further impact as they cannot be retrieved by the searches of any user previously related to the event. 

----

**4.9. Event Lists Sorting**<br>

Lists of events are sorted by ascending distance by default, closer first. Besides, the system must also allow them to be sorted by the number of topics in the user's "interests" or the number of friends who selected ‘interested’ or ‘going’. 

----

**4.10. Map Search**<br>

The system also allows users to search on a map by opening a map, showing the user's current location and the event's location in a distance of up to 10 km. 

----

**4.11. Event Lists Topic Filter**<br>

Lists of events can be filtered by topics, keeping only the events related to specific topics and/or excluding the events ssociated with certain topics. 

**4.12. Map Topic Filter**<br>

Events displayed on the map can be filtered by topics, keeping only the events related to specific topics and/or excluding the events associated with certain topics. 

----

**4.13. Event Lists Time Filter**<br>

Lists of events can be filtered by date by selecting a time window (specifying start and end datetime) and showing only the events happening within this window. 

----

**4.14. Map Time Filter**<br>

Events displayed on the map can be filtered by date by selecting a time window (specifying start and end datetime) and showing only the events happening within this window. 

----

**4.15. Map Events Information**<br>

When clicking on an event tag in the map, an information pop-up appears with basic info such as the event's name, the ticket price (if any), the location, and a link to the event details page. 

