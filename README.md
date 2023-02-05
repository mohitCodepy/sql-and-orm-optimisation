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
        distinct_cities = YourModel.objects.values('city').annotate(city_count=Count('city')).count()

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
