# sql-and-orm-optimisation
Snippet of sql and orm for bettter understanding

#### To get the difference of all CITY and distinct CITY from STATION table:
- sql:
    ```
        SELECT COUNT(CITY) - COUNT(DISTINCT CITY) FROM STATION;
    ```

- django: 

    Using 2 queries (normal)
    ```
        from django.db.models import Count

        total_cities = YourModel.objects.all().count()
        distinct_cities = YourModel.objects.values('city').annotate(city_count=Count('city', distinct=True)).count()

        difference = total_cities - distinct_cities
    ```

    Using single query:
    ```
        from django.db.models import Count

        result = YourModel.objects.aggregate(
            total_cities=Count('city'),
            distinct_cities=Count('city', distinct=True)
        )

        difference = result['total_cities'] - result['distinct_cities']

    ```

#### To get the difference of maximum and minimum of population from city table:
- sql:
    ```
        SELECT MAX(POPULATION) - MIN(POPULATION) FROM CITY;
    ```

- django:

    Using 2 queries
    ```
        from django.db.models import Max, Min

        max_population = City.objects.aggregate(Max('population'))['population__max']
        min_population = City.objects.aggregate(Min('population'))['population__min']
        difference = max_population - min_population
    ```
    
    Using single query:
    ```
        from django.db.models import Max, Min

        difference = City.objects.aggregate(
            difference=Max('population') - Min('population')
        ).get('difference')
    ```
    
#### To find the avg difference between salary and wrong salary, for example if

| Name   | Mohit   | Aman   |
| :---:  | :---:   | :---:  |
| Salary | 1000    | 2000   |

so the avg salary will be (1000+2000)/2 = 1500

but suppose you want to replace the zero (if your calculator's zero button is not working ;) )
| Name   | Mohit   | Aman   |
| :---:  | :---:   | :---:  |
| Salary | 1    | 2   |

wrong avg will be : (1+2)/2 = 1.5

then you want to find the difference between the correct avg and miscalculated avg then:

- sql:
    ```
        SELECT CEIL(AVG(SALARY) - AVG(REPLACE(SALARY, 0, ''))) FROM EMPLOYEES;
    ```

- django:
    ```
        from django.db.models import Avg, F, FloatField, Func
        from math import ceil

        class Ceil(Func):
            function = 'CEIL'

        error = Employees.objects.aggregate(
            error=Ceil(Avg('salary') - 
                       (Avg(F('salary').replace(0, '').cast(FloatField())))),
        ).get('error')
    ```

#### To find the values with some concatinated string

- sql:
    ```
        SELECT 
            CONCAT(Name, "(",LEFT(Occupation,1),")")
        FROM
            OCCUPATIONS
        ORDER BY
            Name;

        SELECT 
            CONCAT("There are total of ", COUNT(Occupation), " ", LOWER(Occupation), "s.")
        FROM OCCUPATIONS
        GROUP BY
            Occupation
        ORDER BY
            COUNT(*) ASC, Occupation ASC;
    ```

- django:
```
        from django.db.models import Value, CharField
        from django.db.models.functions import Concat, Substr

        Occupations.objects.annotate(
            new_name=Concat(
                'name', 
                Value('('), 
                Substr('occupation', 1, 1),
                Value(')'),
                output_field=CharField()
            )
        ).values_list('new_name', flat=True).order_by('name')
     
        Occupations.objects.values('occupation') \
            .annotate(count=Count('occupation')) \
            .annotate(
                new_occupation=Concat(
                    Value('There are total of '), 
                    'count', 
                    Value(' '), 
                    Lower('occupation'), 
                    Value('s.'),
                    output_field=CharField()
                )
            ).order_by('count', 'occupation').values_list('new_occupation', flat=True)
```

#### Add the event cancellation fees of the all users wallet amount 

- sql
```commandline
UPDATE 
  "testapp_userwallet" 
SET 
  "wallet" = (
    "testapp_userwallet"."wallet" + (
      SELECT 
        U2."fee" 
      FROM 
        "testapp_userwallet" U0 
        INNER JOIN "testapp_user" U1 ON (U0."user_id" = U1."id") 
        LEFT OUTER JOIN "testapp_eventhall" U2 ON (U1."id" = U2."user_id") 
      WHERE 
        U0."id" = ("testapp_userwallet"."id") 
      LIMIT 
        1
    )
  ) 
WHERE 
  "testapp_userwallet"."id" IN (
    SELECT 
      V0."id" 
    FROM 
      "testapp_userwallet" V0 
      INNER JOIN "testapp_user" V1 ON (V0."user_id" = V1."id") 
      INNER JOIN "testapp_eventhall" V2 ON (V1."id" = V2."user_id") 
    WHERE 
      V2."event_id" = 1
  )

```

- django
```commandline
UserWallet.objects.filter(user_event_userevent_id=1).update(
    wallet=F("wallet")
    + Subquery(
        UserWallet.objects.filter(pk=OuterRef("pk")).values("userevent_user_fee")[:1]
    )
)

```

#### Get the count of events in a city according to the event type

- sql
```commandline
SELECT 
  COUNT("testapp_event"."event") FILTER (
    WHERE 
      "testapp_event"."event_type" = \ 'message.bounced\') AS "bounced", COUNT("testapp_event"."event") FILTER (WHERE "testapp_event"."event_type" = \'message.created\') AS "created", COUNT("testapp_event"."event") FILTER (WHERE "testapp_event"."event_type" = \'message.replied\') AS "replied" FROM "testapp_event"

```

- django
```commandline
events.aggregate(
    bounced=Count("event", filter=Q(event_type="message.bounced")),
    created=Count("event", filter=Q(event_type="message.created")),
    replied=Count("event", filter=Q(event_type="message.replied")),
)

```