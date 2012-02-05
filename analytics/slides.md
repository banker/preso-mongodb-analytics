!SLIDE

# Analytics with MongoDB

!SLIDE

## Kyle Banker
### @hwaet

!SLIDE bullets incremental

### Challenges:

 * Real-time reporting.
 * Efficient use of space.
 * Easily wipe unneeded data.

!SLIDE bullets incremental

### Techniques

 * Pre-aggregate the data.
 * Pre-construct document structure.
 * Store each collection in its own database.

!SLIDE bullets incremental

### Two collections:

 * Collections names are time-scoped.
 * Clean, fast removal of old data by dropping associated database.

!SLIDE small

    @@@ JavaScript
    // Collections holding totals for each day, stored
    // in a database per month
    days_2011_5
    days_2011_6
    days_2011_7
    ...

    // Totals for each month...
    months_2011_1_4
    months_2011_5_8
    months_2011_9_12
    ...

!SLIDE smaller

### Hours and minutes

    @@@ JavaScript
    { _id: { uri: BinData("0beec7b5ea3f0fdbc95d0dd47f35"),
             day: '2011-5-1'
           },
      total: 2820,
      hrs: { 0: 500,
             1: 700,
             2: 450,
             3: 343,
             // ... 4-23 go here
             }
       // Minutes are rolling. This gives real-time
       // numbers for the last hour. So when you increment
       // minute n, you need to $set minute n-1 to 0.
       mins: { 1: 12,
               2: 10,
               3: 5,
               4: 34
               // ... 5-60 go here
             }
    }

!SLIDE green

## Updating the day totals document...

!SLIDE smaller

    @@@ JavaScript
    // Update 'days' collection at 5:37 p.m. on 2011-5-1
    // Might want to queue increments so that $inc is greater
    // than 1 for each write...
    id = { uri: BinData("0beec7b5ea3f0fdbc95d0dd47f35"),
           day: '2011-5-1'
         };

    update = { $inc: { total: 1, 'hrs.17': 1, 'mins.37': 1 },
               $set: { 'mins.36': 0} };

    // Update collection containing days 1-5
    db.days_2011_5.update( id, update );

!SLIDE smaller

### Months and days

    @@@ JavaScript
    { _id: { uri: BinData("0beec7b5ea3f0fdbc95d0dd47f35"),
             month: '2011-5'
           },
      total: 34173,
      months: { 1: 4000,
                2: 4300,
                3: 4200,
                4: 5000,
                5: 5100,
                6: 5700,
                7: 5873
                // ... 8-12 go here
             }
    }

!SLIDE green

## Updating month document...

!SLIDE smaller

    @@@ JavaScript
    // Update 'months' collection at 5:37 p.m. on 2011-5-1
    query = { uri: BinData("0beec7b5ea3f0fdbc95d0dd47f35"),
              month: '2011-5'
            };

    update = { $inc: { total: 1, 'months.5': 1 } };

    // Update collection containing days 1-5
    db.month_2011_1_4.update( query, update );

!SLIDE bullets incremental

## Reporting

 * Must provide data at multiple resolutions (second, minute, etc.).
 * We have the raw materials for that.
 * Application assembles the data intelligently.

!SLIDE

# FIN
