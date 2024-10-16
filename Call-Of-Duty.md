# Call Of Duty

## **Contents**

1. **Tools**
   - **[SQL Queries](#sql-queries)**
     - Find point by provider_id
     - Find locker_cells by provider_id
2. **How to**
   - **[Create a new city](#create-a-new-city)**
   - **[Import BloqIt point to Vcarrier](#import-bloqit-point-to-vcarrier)**
   - **[Unchurn point](#unchurn-point)**
   - **[Sync Hubspot <-> Vcarrier <-> BloqIt](#sync-hubspot-vcarrier-bloqit)**
   - **[Switch locker into a shop](#switch-locker-into-a-shop)**
   - **[Switch shop into a locker](#switch-shop-into-a-locker)**
   - **[Change locker `provider_id`](#change-locker-provider_id)**
   - **[Hide point until date (used for spiking points)](#hide-point-until-date-used-for-spiking-points)**

## Tools

### SQL Queries

```SQL
-- Find point by provider_id
SELECT p.id, p.point_type, p.provider_id, p.code, p.name, p.country_code, p.external_id, p.locale
FROM `points` as p
WHERE p.provider_id IN ('640f9fa75ed7b08880604142', '64a3626cb640335b44569520', '66d74aa8751dfc1a370b6a09', '66e87462daabd3c106954988')

-- Find locker_cells by provider_id
SELECT *
FROM locker_cells as lc
WHERE lc.provider_id = '66e87462daabd3c10695499c'
```

## How to

### Create a new city

- [Confluence - Create a new city](https://vinted.atlassian.net/wiki/spaces/racon/pages/28624355366/How-to+create+a+city)
- [Github/svc-vcarrier - Rake task: create new city](https://github.com/vinted/svc-vcarrier/blob/3fde2c5e239fa3c5aa53e0533de97614c553204d/lib/tasks/cities.rake#L10)

```Shell
# Create cities [Paris,Lille,'Les Pavillons']
@mixer rake svc-vcarrier/rake prod3 cities:create -- -d Paris,Lille,'Les Pavillons' -c 'FR'
# Example
@mixer rake svc-vcarrier/rake prod3 cities:create -- -d "Villeneuve-d'Ascq" -c 'FR'
# 7 Bd de Mons, 59650 Villeneuve-d'Ascq, France
```

```Shell
# Options
Usage: cities:create [options]
    -c, --country_code {country_code}     Country code. If not provided it will be set to default
    -d, --cities {cities}                 List of cities for given country
    -h, --help                            Instructions
```

### Import BloqIt point to Vcarrier

```Shell
# Import
@mixer rake svc-vcarrier/rake prod3 points:create_update_point['<bloq_id>']
# Example
@mixer rake svc-vcarrier/rake prod3 points:create_update_point['66868866149e49e4574ba636']
```

### Unchurn point

```Shell
# Unchurn
@mixer rake svc-vcarrier/rake prod3 points:undo_churn[point_id,full_restore]
# Example
@mixer rake svc-vcarrier/rake prod3 points:undo_churn[105258]
```

### Sync Hubspot <-> Vcarrier <-> BloqIt

### Switch locker into a shop

1. [x] Check if locker doesn't exist in BloqIt as locker or Location
2. [x] In Vcarrier, check if locker exists with the expected `external_id`
3. [ ] In Vcarrier, change locker's `Point.external_id` to `<external_id>-1` with the following rake task:

    ```Shell
    # Command
    @mixer rake svc-vcarrier/rake prod3 points:update_external_id['<external_id>','<new_external_id>']
    # Example
    @mixer rake svc-vcarrier/rake prod3 points:update_external_id['12191862471','12191862471-1']
    ```

4. [ ] Check if changes worked:
   1. [ ] Old `external_id` doesn't exist anymore
   2. [ ] The old point bears the new `<external_id>-1`

### Switch shop into a locker

1. [ ] In Vcarrier, change locker's `Point.external_id` to `<external_id>-1` with the following rake task:

    ```Shell
    @mixer rake svc-vcarrier/rake prod3 points:update_external_id['<external_id>','<new_external_id>']
    # Example
    @mixer rake svc-vcarrier/rake prod3 points:update_external_id['12191862471','12191862471-1']
    ```

2. [ ] Check if change works:
   1. [ ] Old `external_id` doesn't exist anymore
   2. [ ] The old point bears the new `<external_id>-1`

### Change locker `provider_id`

1. [ ] Check if locker exists
2. [ ] Check if locker's cell are empty
3. [ ] Execute rake

    ```Shell
    @mixer rake svc-vcarrier/rake prod3 points:update_provider_id['<point_id>','<provider_id>']
    # Example
    @mixer rake svc-vcarrier/rake prod3 points:update_provider_id['00200','66f2cb7e8c9623b6450437e0']
    @mixer rake svc-vcarrier/rake prod3 points:update_provider_id['00363','66f2cb7f9589a32519905985']
    ```

4. [ ] Check if changes worked
   1. [ ] In Vcarrier
   2. [ ] In BloqIt

### Hide point until date (used for spiking points)

1. [ ] Download a snapshot of `points` table before execution:

    ```SQL
    SELECT *
    FROM `points` as p
    WHERE p.id IN (342,102222,106494,100190,102522,102106,101900,107570,104621,100427,20045,322,
        434,10049,69,106022,105,103891,106963,103240,29,106490,30010,103174,100877,
        104382,266,106954,106210,393,321,326,102179,100176,164,106573,104088,338,289,
        101368,100626,106366)
    ```

2. [ ] Upload `point_id` on GithubGist
3. [ ] Run the rake task
   - [ ] <b style="color:#ffad33">❗️Update duration</b>
   - [ ] <b style="color:#ffad33">❗️Update the GithubGits</b>

    ```Shell
    # Command
    @mixer rake svc-vcarrier/rake prod3 points:hide_points_until_by_point_list[<github_gist>,<hidden_reason>,<duration_in_days>]
    # Example
    @mixer rake svc-vcarrier/rake prod3 points:hide_points_until_by_point_list['https://gist.githubusercontent.com/remigyus/8ce2239e800fac74aee2405807c43557/raw/5c32b89e03125a0d67e31cfc11413da4b4fa43b1/hide_point_for_tx_16_oct.csv','parcel_backlog',1]
    ```

4. [ ] Download a snapshot of `points` table's final state
