# Build LookML Objects in Looker
[Looker Overview](https://cloud.google.com/looker/docs)
## Looker Developer: Qwik Start [GSP891]

LookML hierarchy
Project
  Model
    Explore
      View

### Create a view
toggle Development Mode bottom
Develop
qwiklabs-ecommerce
+
Create View
users_limited
Create
views
drag into views tree

sql_table_name: `cloud-training-demos.looker_ecomm.users` ;;

dimension: id {
  primary_key: yes
  type: number
  sql: ${TABLE}.id ;;
}

dimension: country {
  type: string
  map_layer_name: countries
  sql: ${TABLE}.country ;;
}

dimension: email {
  type: string
  sql: ${TABLE}.email ;;
}

dimension: first_name {
  type: string
  sql: ${TABLE}.first_name ;;
}

dimension: last_name {
  type: string
  sql: ${TABLE}.last_name ;;
}

measure: count {
  type: count
  drill_fields: [id, last_name, first_name]
}

Save Changes
Validate LookML
Commit Changes & Push
Commit
Deploy to Production

### Join a view to an existing Explore
models
training_ecommerce.model

join: users_limited {
  type: left_outer
  sql_on: ${events.user_id} = ${users_limited.id};;
  relationship: many_to_one
}

Save Changes
Explore Events
Users Limited
First Name
Count
Run
navigate back to the training_ecommerce.model
Validate LookML
Commit Changes & Push
Commit
Deploy to Production

## Creating Measures and Dimensions Using LookML [GSP890]
###  Creating dimensions
toggle Development mode
Develop
qwiklabs-ecommerce
views
users.view
under age

dimension: age_tier {
  type: tier
  tiers: [18, 25, 35, 45, 55, 65, 75, 90]
  style: integer
  sql: ${age} ;;
}

Save Changes 
Validate LookML
Explore Order Items
Users
Dimensions
Age Tier
add Age and Age Tier
Run

users.view
look for traffic_source

dimension: is_email_source {
  type: yesno
  sql: ${traffic_source} = "Email" ;;
}

order_items.view
find shipped

dimension: shipping_days {
  type: number
  sql: DATE_DIFF(${shipped_date}, ${created_date}, DAY);;
}

Save Changes
Validate LookML
Commit Changes & Push
Commit
Deploy to Production

###  Creating measures
qwiklabs-ecommerce
order_items.view
under order_item_count

measure: count_distinct_orders {
  type: count_distinct
  sql: ${order_id} ;;
}

measure: total_sales {
  type: sum
  sql: ${sale_price} ;;
  value_format_name: usd_0
}

Save Changes
Validate LookML
Commit Changes & Push
Commit
Deploy to Production

###  Creating advanced measures
qwiklabs-ecommerce
order_items.view
under order_item_count

measure: total_sales_email_users {
  type: sum
  sql: ${sale_price} ;;
  filters: [users.is_email_source: "Yes"]
}

or

measure: total_sales_email_users {
  type: sum
  sql: ${sale_price} ;;
  filters: [users.traffic_source: "Email"]
}

measure: percentage_sales_email_source {
  type: number
  value_format_name: percent_2
  sql: 1.0*${total_sales_email_users}
  / NULLIF(${total_sales}, 0) ;;
}

Save Changes
Validate LookML
Commit Changes & Push
Commit
Deploy to Production

## Creating Derived Tables Using LookML [GSP858]
### Create a SQL derived table summarizing details for each order
- Define a new derived table using a SQL query
SELECT
  order_items.order_id AS order_id
  ,order_items.user_id AS user_id
  ,COUNT(*) AS order_item_count
  ,SUM(order_items.sale_price) AS order_revenue
FROM cloud-training-demos.looker_ecomm.order_items
GROUP BY order_id, user_id

Settings
Add to Project
qwiklabs-ecommerce
order_details
Add

Save Changes
Validate LookML
Commit Changes & Push
Commit
Deploy to Production

### Create a native derived table summarizing details for each order
Looker navigation menu
Explore
E-Commerce Training
Order Items
Order Items

Order ID
User ID
Order Count
Total Revenue

Run
Settings
Get LookML
Derived Table

Looker navigation menu
Develop
+
Create View
order_details_summary
Create
replace view: order_details_summary {}

Save Changes
Validate LookML
Commit Changes & Push
Commit
Deploy to Production

### Join a new view to an Explore
models
training_ecommerce.model
under users [join]

join: order_details {
    type: left_outer
    sql_on: ${order_items.order_id} = ${order_details.order_id};;
    relationship: many_to_one
  }
Save Changes
Validate LookML
Commit Changes & Push
Commit
Deploy to Production

### Persist a derived table
Looker navigation menu
Develop
views
order_details_summary.view
under explore_source {}

datagroup_trigger: training_ecommerce_default_datagroup

Save Changes
Validate LookML
Commit Changes & Push
Commit
Deploy to Production

## Filtering Explores with LookML [GSP892]
### Types of Explore filters
To filter an Explore, you need to apply a default WHERE or HAVING clause to every SQL query that gets generated in that Explore. There are three principal ways to filter an Explore:

sql_always_where and sql_always_having, which behave similarly and have the same use case
always_filter
conditionally_filter
In the following sections, you learn about common use cases for each of these.

The sql_always_where and sql_always_having filters
Both sql_always_where and sql_always_having allow you to add filters to an Explore that cannot be modified. This is useful when you have certain rows of data you always want to exclude from the Explore results.

The sql_always_where filter is used to add a WHERE clause applied to dimensions in a SQL query, whereas sql_always_having is used to add a HAVING clause applied to measures in a SQL query. In addition to queries run explicitly by business users, the restriction will apply to dashboards, scheduled Looks, and embedded information that relies on that Explore.

There will be no indication of the filter in the user interface, so business users are not informed that the data are being filtered, unless they have permission to look at the generated SQL. This is useful if you want to filter out certain values of the Explore, such as test or internal data.

The always_filter
The always_filter enables you to require users to include a certain set of filters that you define. You also define a default value for the filters. Though users may change your default value for their query, they cannot remove the filter entirely. This is helpful when you want users to always filter by specific dimensions, such as always filtering by order status or user country, so that they do not request all of the possible data at one time.

The always_filter has a sub-parameter to define the specific filters using the same Looker filter expressions that are used to filter dimensions and measures. The dimensions provided in the filters sub-parameter identify the dimensions that users must provide values for, such as a value for order status or user country.

The specific values provided for in the filters sub-parameter are the default values which can be changed by the business user. For example, while the default order status is “Complete”, business users can change this value to say orders with a different status like “Returned”. For additional information, review the Looker filter expressions document.

The conditionally_filter
Similar to the always_filter, the conditionally_filter adds a filter to the Explore frontend that is accessible by business users. The conditionally_filter parameter enables you to define a set of default filters that users can override if they apply at least one filter from a second list that you define.

Although users can indeed change the filter operator and values, they cannot remove the filter itself unless they put a filter on a specific alternative field. This is helpful when you want to limit the amount of data that an business user requests, but you also want to give them a list of alternative dimensions that they can use to filter the data.

Conditionally_filter has a sub-parameter to define the specific filters as well as a sub-parameter to define the alternative dimensions that can be used to filter the data. For example, conditionally_filter can be used to create a filter that only returns data for the past 1 year, unless a filter is applied to a user ID or state dimension. This is typically used to prevent users from accidentally creating very large queries that may be too expensive to run on your database.

### Add an always_filter
develop
qwiklabs_ecommerce
training_ecommerce.model

under explore: order_items

always_filter: {
  filters: [order_items.status: "Complete", users.country: "USA"]
}

```
connection: "bigquery_public_data_looker"

# include all the views
include: "/views/*.view"
include: "/z_tests/*.lkml"
include: "/**/*.dashboard"

datagroup: training_ecommerce_default_datagroup {
  # sql_trigger: SELECT MAX(id) FROM etl_log;;
  max_cache_age: "1 hour"
}

persist_with: training_ecommerce_default_datagroup

label: "E-Commerce Training"

explore: order_items {
  #####
  # add line here
  always_filter: {
  filters: [order_items.status: "Complete", users.country: "USA"]
  }
  #####
  join: users {
    type: left_outer
    sql_on: ${order_items.user_id} = ${users.id} ;;
    relationship: many_to_one
  }

  join: inventory_items {
    type: left_outer
    sql_on: ${order_items.inventory_item_id} = ${inventory_items.id} ;;
    relationship: many_to_one
  }

  join: products {
    type: left_outer
    sql_on: ${inventory_items.product_id} = ${products.id} ;;
    relationship: many_to_one
  }

  join: distribution_centers {
    type: left_outer
    sql_on: ${products.distribution_center_id} = ${distribution_centers.id} ;;
    relationship: many_to_one
  }
}

explore: events {
  join: event_session_facts {
    type: left_outer
    sql_on: ${events.session_id} = ${event_session_facts.session_id} ;;
    relationship: many_to_one
  }
  join: event_session_funnel {
    type: left_outer
    sql_on: ${events.session_id} = ${event_session_funnel.session_id} ;;
    relationship: many_to_one
  }
  join: users {
    type: left_outer
    sql_on: ${events.user_id} = ${users.id} ;;
    relationship: many_to_one
  }
}
```

### Add a sql_always_where filter
develop
qwiklabs_ecommerce
training_ecommerce.model

under explore: order_items

sql_always_where: ${created_date} >= '2021-01-01' ;;

```
connection: "bigquery_public_data_looker"

# include all the views
include: "/views/*.view"
include: "/z_tests/*.lkml"
include: "/**/*.dashboard"

datagroup: training_ecommerce_default_datagroup {
  # sql_trigger: SELECT MAX(id) FROM etl_log;;
  max_cache_age: "1 hour"
}

persist_with: training_ecommerce_default_datagroup

label: "E-Commerce Training"

explore: order_items {
  #####
  # add line here
  sql_always_where: ${created_date} >= '2021-01-01' ;;
  #####
  join: users {
    type: left_outer
    sql_on: ${order_items.user_id} = ${users.id} ;;
    relationship: many_to_one
  }

  join: inventory_items {
    type: left_outer
    sql_on: ${order_items.inventory_item_id} = ${inventory_items.id} ;;
    relationship: many_to_one
  }

  join: products {
    type: left_outer
    sql_on: ${inventory_items.product_id} = ${products.id} ;;
    relationship: many_to_one
  }

  join: distribution_centers {
    type: left_outer
    sql_on: ${products.distribution_center_id} = ${distribution_centers.id} ;;
    relationship: many_to_one
  }
}

explore: events {
  join: event_session_facts {
    type: left_outer
    sql_on: ${events.session_id} = ${event_session_facts.session_id} ;;
    relationship: many_to_one
  }
  join: event_session_funnel {
    type: left_outer
    sql_on: ${events.session_id} = ${event_session_funnel.session_id} ;;
    relationship: many_to_one
  }
  join: users {
    type: left_outer
    sql_on: ${events.user_id} = ${users.id} ;;
    relationship: many_to_one
  }
}
```

### Add a sql_always_having filter
develop
qwiklabs_ecommerce
training_ecommerce.model

under explore: order_items

sql_always_having: ${order_item_count} = 1 ;;

```
connection: "bigquery_public_data_looker"

# include all the views
include: "/views/*.view"
include: "/z_tests/*.lkml"
include: "/**/*.dashboard"

datagroup: training_ecommerce_default_datagroup {
  # sql_trigger: SELECT MAX(id) FROM etl_log;;
  max_cache_age: "1 hour"
}

persist_with: training_ecommerce_default_datagroup

label: "E-Commerce Training"

explore: order_items {
  #####
  # add line here
  sql_always_having: ${order_item_count} = 1 ;;
  #####
  join: users {
    type: left_outer
    sql_on: ${order_items.user_id} = ${users.id} ;;
    relationship: many_to_one
  }

  join: inventory_items {
    type: left_outer
    sql_on: ${order_items.inventory_item_id} = ${inventory_items.id} ;;
    relationship: many_to_one
  }

  join: products {
    type: left_outer
    sql_on: ${inventory_items.product_id} = ${products.id} ;;
    relationship: many_to_one
  }

  join: distribution_centers {
    type: left_outer
    sql_on: ${products.distribution_center_id} = ${distribution_centers.id} ;;
    relationship: many_to_one
  }
}

explore: events {
  join: event_session_facts {
    type: left_outer
    sql_on: ${events.session_id} = ${event_session_facts.session_id} ;;
    relationship: many_to_one
  }
  join: event_session_funnel {
    type: left_outer
    sql_on: ${events.session_id} = ${event_session_funnel.session_id} ;;
    relationship: many_to_one
  }
  join: users {
    type: left_outer
    sql_on: ${events.user_id} = ${users.id} ;;
    relationship: many_to_one
  }
}
```
### Add a conditional_filter
develop
qwiklabs_ecommerce
training_ecommerce.model

under explore: order_items

conditionally_filter: {
  filters: [created_date: "3 years"]
  unless: [users.id, users.state]
}

```
connection: "bigquery_public_data_looker"

# include all the views
include: "/views/*.view"
include: "/z_tests/*.lkml"
include: "/**/*.dashboard"

datagroup: training_ecommerce_default_datagroup {
  # sql_trigger: SELECT MAX(id) FROM etl_log;;
  max_cache_age: "1 hour"
}

persist_with: training_ecommerce_default_datagroup

label: "E-Commerce Training"

explore: order_items {
  #####
  # add line here
  conditionally_filter: {
  filters: [created_date: "3 years"]
  unless: [users.id, users.state]
  }
  #####
  join: users {
    type: left_outer
    sql_on: ${order_items.user_id} = ${users.id} ;;
    relationship: many_to_one
  }

  join: inventory_items {
    type: left_outer
    sql_on: ${order_items.inventory_item_id} = ${inventory_items.id} ;;
    relationship: many_to_one
  }

  join: products {
    type: left_outer
    sql_on: ${inventory_items.product_id} = ${products.id} ;;
    relationship: many_to_one
  }

  join: distribution_centers {
    type: left_outer
    sql_on: ${products.distribution_center_id} = ${distribution_centers.id} ;;
    relationship: many_to_one
  }
}

explore: events {
  join: event_session_facts {
    type: left_outer
    sql_on: ${events.session_id} = ${event_session_facts.session_id} ;;
    relationship: many_to_one
  }
  join: event_session_funnel {
    type: left_outer
    sql_on: ${events.session_id} = ${event_session_funnel.session_id} ;;
    relationship: many_to_one
  }
  join: users {
    type: left_outer
    sql_on: ${events.user_id} = ${users.id} ;;
    relationship: many_to_one
  }
}
```
## Build LookML Objects in Looker: Challenge Lab [GSP361]
### Create dimensions and measures
- 

### Create a persistent derived table
- 

### Use Explore filters
- 

### Apply a datagroup to an Explore
- 

[SOLUTION](https://github.com/quiccklabs/Labs_solutions/blob/6919838bdee340ac97f3d308d341758c1468806e/Build%20LookML%20Objects%20in%20Looker%3A%20Challenge%20Lab)

FILE NAME :- order_items_challenge


view: order_items_challenge {
  sql_table_name: `cloud-training-demos.looker_ecomm.order_items’  ;;
  drill_fields: [order_item_id]
  dimension: order_item_id {
    primary_key: yes
    type: number
    sql: ${TABLE}.id ;;
  }

  dimension: is_search_source {
    type: yesno
    sql: ${users.traffic_source} = "Search" ;;
  }


  measure: sales_from_complete_search_users {
    type: sum
    sql: ${TABLE}.sale_price ;;
    filters: [is_search_source: "Yes", order_items.status: "Complete"]
  }


  measure: total_gross_margin {
    type: sum
    sql: ${TABLE}.sale_price - ${inventory_items.cost} ;;
  }


  dimension: return_days {
    type: number
    sql: DATE_DIFF(${order_items.delivered_date}, ${order_items.returned_date}, DAY);;
  }
  dimension: order_id {
    type: number
    sql: ${TABLE}.order_id ;;
  }

}





=========================================================================================================================================================




FILE NAME :- user_details


# If necessary, uncomment the line below to include explore_source.
# include: "training_ecommerce.model.lkml"

view: user_details {
  derived_table: {
    explore_source: order_items {
      column: order_id {}
      column: user_id {}
      column: total_revenue {}
      column: age { field: users.age }
      column: city { field: users.city }
      column: state { field: users.state }
    }
  }
  dimension: order_id {
    description: ""
    type: number
  }
  dimension: user_id {
    description: ""
    type: number
  }
  dimension: total_revenue {
    description: ""
    value_format: "$#,##0.00"
    type: number
  }
  dimension: age {
    description: ""
    type: number
  }
  dimension: city {
    description: ""
  }
  dimension: state {
    description: ""
  }
}





=========================================================================================================================================================




FILE NAME :- training_ecommerce




connection: "bigquery_public_data_looker"

# include all the views
include: "/views/*.view"
include: "/z_tests/*.lkml"
include: "/**/*.dashboard"

datagroup: training_ecommerce_default_datagroup {
  # sql_trigger: SELECT MAX(id) FROM etl_log;;
  max_cache_age: "1 hour"
}

persist_with: training_ecommerce_default_datagroup

label: "E-Commerce Training"

explore: order_items {



  sql_always_where: ${sale_price} >= VALUE_1 ;;


  conditionally_filter: {

    filters: [order_items.shipped_date: "2018"]

    unless: [order_items.status, order_items.delivered_date]

  }


  sql_always_having: ${average_sale_price} > VALUE_2 ;;

  always_filter: {
    filters: [order_items.status: "Shipped", users.state: "California", users.traffic_source:
      "Search"]
  }



  join: user_details {

    type: left_outer

    sql_on: ${order_items.user_id} = ${user_details.user_id} ;;

    relationship: many_to_one

  }


  join: order_items_challenge {
    type: left_outer
    sql_on: ${order_items.order_id} = ${order_items_challenge.order_id} ;;
    relationship: many_to_one
  }

  join: users {
    type: left_outer
    sql_on: ${order_items.user_id} = ${users.id} ;;
    relationship: many_to_one
  }



  join: inventory_items {
    type: left_outer
    sql_on: ${order_items.inventory_item_id} = ${inventory_items.id} ;;
    relationship: many_to_one
  }

  join: products {
    type: left_outer
    sql_on: ${inventory_items.product_id} = ${products.id} ;;
    relationship: many_to_one
  }

  join: distribution_centers {
    type: left_outer
    sql_on: ${products.distribution_center_id} = ${distribution_centers.id} ;;
    relationship: many_to_one
  }
}

explore: events {
  join: event_session_facts {
    type: left_outer
    sql_on: ${events.session_id} = ${event_session_facts.session_id} ;;
    relationship: many_to_one
  }
  join: event_session_funnel {
    type: left_outer
    sql_on: ${events.session_id} = ${event_session_funnel.session_id} ;;
    relationship: many_to_one
  }
  join: users {
    type: left_outer
    sql_on: ${events.user_id} = ${users.id} ;;
    relationship: many_to_one
  }
}






==========================================================================================================================================================================



FINAL TASK ::

FILE NAME :-  training_ecommerce




connection: "bigquery_public_data_looker"

# include all the views
include: "/views/*.view"
include: "/z_tests/*.lkml"
include: "/**/*.dashboard"

datagroup: order_items_challenge_datagroup {
  sql_trigger: SELECT MAX(order_item_id) from order_items ;;
  max_cache_age: " hours"
}


persist_with: order_items_challenge_datagroup


label: "E-Commerce Training"

explore: order_items {
  join: user_details {

    type: left_outer

    sql_on: ${order_items.user_id} = ${user_details.user_id} ;;

    relationship: many_to_one

  }


  join: order_items_challenge {
    type: left_outer
    sql_on: ${order_items.order_id} = ${order_items_challenge.order_id} ;;
    relationship: many_to_one
  }

  join: users {
    type: left_outer
    sql_on: ${order_items.user_id} = ${users.id} ;;
    relationship: many_to_one
  }



  join: inventory_items {
    type: left_outer
    sql_on: ${order_items.inventory_item_id} = ${inventory_items.id} ;;
    relationship: many_to_one
  }

  join: products {
    type: left_outer
    sql_on: ${inventory_items.product_id} = ${products.id} ;;
    relationship: many_to_one
  }

  join: distribution_centers {
    type: left_outer
    sql_on: ${products.distribution_center_id} = ${distribution_centers.id} ;;
    relationship: many_to_one
  }
}

explore: events {
  join: event_session_facts {
    type: left_outer
    sql_on: ${events.session_id} = ${event_session_facts.session_id} ;;
    relationship: many_to_one
  }
  join: event_session_funnel {
    type: left_outer
    sql_on: ${events.session_id} = ${event_session_funnel.session_id} ;;
    relationship: many_to_one
  }
  join: users {
    type: left_outer
    sql_on: ${events.user_id} = ${users.id} ;;
    relationship: many_to_one
  }
}




==========================================================================================================================================================================