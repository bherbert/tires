## Design Usage

I discovered Brett Terpstra's "doing" script and thought it might scratch a personal itch; that of keeping tabs on what I'm "doing" regarding the tires going on/off my motorcycle.

So, I cloned the **doing** repo and created **tires**.

The goal is to have an inventory of individual tires, each described by: [brand, purchase-date, UID].  Then, the DSL would allow me to record events against this inventory:

To record the mounting of a particular set of tires:

    > tires on front UID1-front mileage
    > tires on rear UID2-rear mileage

Internally, doing those **on** events would do an automatic corresponding **off** event against the tires that are currently mounted (calculated by scanning the event log for the most-recent **on** event for the specified wheel).

The event log (internal database) would contain a time-ordered list of **on** and **off** events.

    > 2015-04-10, 9:48 AM: ON front UID1-front mileage
    > 2015-04-10, 9:48 AM: ON rear UID1-rear mileage  
    > 2015-04-10, 9:48 AM: OFF front UID1-front mileage
    > 2015-05-10, 5:00 PM: ON front UID2-front mileage
    > 2015-05-10, 5:00 PM: OFF rear UID1-rear mileage
    > 2015-05-10, 5:01 PM: OFF front UID1-front mileage
    > 2015-05-10, 5:01 PM: ON rear UID2-rear mileage

### Statistics

The goal of all this is to track how many miles are on a given tire. 

To learn how many miles are on the currently mounted set:

 	> tires now

	Front: <UID>  mounted: <date>  MilesOn: <miles>
	Read:  <UID>  Mounted: <date>  MilesOn: <miles>

To learn the history of changes for a given tire:

	> tires history <UID>
	
	Tire Change History of: <UID>   TotalMilesOn: <miles>
	   - Mounted:   <date> At: <miles>
	   - Unmounted: <date> At: <miles>
	   - Mounted:   <date> At: <miles>
	   - Unmounted: <date> At: <miles>
	   - Retired: <date>
	
When a tire is being unmounted for the last time and to be retired:

    > tires off rear UID-rear mileage
    > tires retire UID-rear

which would record the following events:

    > 2015-05-10, 5:01 PM: OFF rear UID2-rear mileage
    > 2015-05-10, 5:01 PM: RETIRE rear UID2-rear mileage


	