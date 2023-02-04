# sql-and-orm-optimisation
Snippet of sql and orm for bettter understanding

### To get the difference of all CITY and distinct CITY from STATION table:
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
            total_cities=Count('id'),
            distinct_cities=Count('city', distinct=True)
        )

        difference = result['total_cities'] - result['distinct_cities']

    ```
