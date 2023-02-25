```sql

--ALTER SESSION SET USE_CACHED_RESULT = FALSE;
--Most viewed recipes by day using qualify row_number() to break view ties on same day
     
with events as (
	select
    	event_id,
        session_id,
        event_timestamp,
        parse_json(event_details):recipe_id::string as recipe_id,
        parse_json(event_details):event:string as event
	from events.website_activity
    group by 1,2,3,4,5
),

most_viewed_recipe_by_day as (
	select 
    	date(event_timestamp) as event_day,
        recipe_id,
        count(event_timestamp, recipe_id) as total_views
        --row_number() over (partition by event_day order by total_views desc)
    from events
    where recipe_id is not null
    group by 1,2
   qualify row_number() over (partition by event_day order by total_views desc) = 1
)
 
select * from most_viewed_recipe_by_day
