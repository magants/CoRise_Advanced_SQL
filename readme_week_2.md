### Refactored Query
```sql
     /* refactored to CTE so that I could remove sub query in main select & renamed for easier understanding*/
with food_pref_count as (
         select 
            customer_id,
            count(*) as food_pref_count
         from vk_data.customers.customer_survey
         where is_active = true
         group by 1
),

     /* refactored to CTE so that I could remove sub query in main select & renamed for easier understanding*/
     chicago_geo_loc as (    
        select geo_location /*removed extra line since only one column being selected*/
        from vk_data.resources.us_cities 
        where city_name = 'CHICAGO'
        and state_abbr = 'IL' /*added extra line for "and" statement*/
),

    /* refactored to CTE so that I could remove sub query in main select & renamed for easier understanding*/
    gary_geo_loc as (
        select geo_location /* removed extra line since only one column being selected*/
        from vk_data.resources.us_cities 
        where city_name = 'GARY'
        and state_abbr = 'IN' /* added extra line for "and" statement*/
),

final as (

select 
    first_name || ' ' || last_name as customer_name,
    customer_address.customer_city,
    customer_address.customer_state,
    food_pref_count.food_pref_count,
    (st_distance(us_cities.geo_location, chicago_geo_loc.geo_location) / 1609)::int as chicago_distance_miles,
    (st_distance(us_cities.geo_location, gary_geo_loc.geo_location) / 1609)::int as gary_distance_miles
from vk_data.customers.customer_address /* removed alias in order to use full table name in "select" statement above */
join vk_data.customers.customer_data on /* removed aliases for clarity in join below & dropped join condition 1 line*/
    customer_address.customer_id = customer_data.customer_id 
left join vk_data.resources.us_cities on /*removed alias "us" for clarity below & dropped join condition 1 line */
    upper(rtrim(ltrim(customer_address.customer_state))) = upper(trim(us_cities.state_abbr)) /* indenting & used lower case "upper" & "trim" */
    and trim(lower(customer_address.customer_city)) = trim(lower(us_cities.city_name))
inner join food_pref_count on customer_data.customer_id = food_pref_count.customer_id /* added "inner" & removed alias for clarity in select & joins */
cross join chicago_geo_loc
cross join gary_geo_loc
where        
        (lower(customer_state) = 'ky' and lower(trim(customer_city)) in ('concord','georgetown','ashland'))
     or (lower(customer_state) = 'ca' and lower(trim(customer_city)) in ('oakland','pleasant hill'))
     or (lower(customer_state) = 'tx' and lower(trim(customer_city)) in ('arlington','brownsville')) 
     -- added brownsville to 'tx' condition above. think original query had mistake after data showed many brownsvilles that didn't return in original result set.
)
            
select * from final
