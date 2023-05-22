## SQL Optimization

### Issue

There are a `SQL` query that produces a search result. The query produces results in approximately 8 seconds.

### Purpose
To suggest improvements that can be done to the query in order to improve its
performance.

---

### Total Hour Spent
+- 3 Hours

### Suggestion Improvements

**1. Set Indexes**
* Set and use index for columns that used for filter and join the relation tables
```
	ALTER TABLE JobCategories
	ADD INDEX name (name);
```
**2. Specify related columns**
* Limit the number of columns or use only related columns for `SELECT` function. Especially for columns that have long text. 
* Minimum column during fetching query result to better performance. Some of the column maybe used for filtering or sorting and not for display purpose.
> **Note:** If not use in the result list, can ignore the following columns: 
```
 1. created_by
 2. created
 3. modified
 4. deleted
 5. sort_order
 6. seo_description
 7. seo_keywords
```

**3. Specify column with its table name when `JOIN` tables**
* Set the column by its table name or its alias
> **Note:** From the query, column status in `WHERE` clause not specify the table name. So it can cause to query error for some of SQL versions.
```
AND Jobs.publish_status = 1
```

**4. Use end wildcard for `LIKE` Operator**
* Use end wildcard as `ABC%` to filter columns which start with a specific value instead of using both start and end wildcard `%ABC%` to search value from any position.
```
WHERE ((JobCategories.name LIKE 'キャビンアテンダント%'
OR JobTypes.name LIKE 'キャビンアテンダント%'
OR Jobs.name LIKE 'キャビンアテンダント%'
OR Jobs.description LIKE 'キャビンアテンダント%'
OR Jobs.detail LIKE 'キャビンアテンダント%'
OR Jobs.business_skill LIKE 'キャビンアテンダント%'
OR Jobs.knowledge LIKE 'キャビンアテンダント%'
OR Jobs.location LIKE 'キャビンアテンダント%'
OR Jobs.activity LIKE 'キャビンアテンダント%'
OR Jobs.salary_statistic_group LIKE 'キャビンアテンダント%'
OR Jobs.salary_range_remarks LIKE 'キャビンアテンダント%'
OR Jobs.restriction LIKE 'キャビンアテンダント%'
OR Jobs.remarks LIKE 'キャビンアテンダント%'
OR Personalities.name LIKE 'キャビンアテンダント%'
OR PracticalSkills.name LIKE 'キャビンアテンダント%'
OR BasicAbilities.name LIKE 'キャビンアテンダント%'
OR Tools.name LIKE 'キャビンアテンダント%'
OR CareerPaths.name LIKE 'キャビンアテンダント%'
OR RecQualifications.name LIKE 'キャビンアテンダント%'
OR ReqQualifications.name LIKE 'キャビンアテンダント%')
AND Jobs.publish_status = 1
AND (Jobs.deleted) IS NULL)
```

**5. Minimize `OR` usage as filtering**
* Replace the `OR` with `UNION` usage for same table filtering
> **Note:** For table `jobs` in the query, there are too many `OR` usage from the same table in filtering which is lead to poor performance. Can use `UNION` instead because it can filter in a combination of tables.
```
WHERE ((JobCategories.name LIKE 'キャビンアテンダント%'
OR JobTypes.name LIKE 'キャビンアテンダント%'
OR Jobs.id IN 
(
	SELECT j1.id FROM jobs j1 WHERE j1.name LIKE 'キャビンアテンダント%' AND j1.publish_status = 1 AND (j1.deleted) IS NULL 
	UNION
	SELECT j2.id FROM jobs j2 WHERE j2.description LIKE 'キャビンアテンダント%' AND j2.publish_status = 1 AND (j2.deleted) IS NULL 
	UNION
	SELECT j3.id FROM jobs j3 WHERE j3.detail LIKE 'キャビンアテンダント%' AND j3.publish_status = 1 AND (j3.deleted) IS NULL 
	UNION
	SELECT j4.id FROM jobs j4 WHERE j4.business_skill LIKE 'キャビンアテンダント%' AND j4.publish_status = 1 AND (j4.deleted) IS NULL 
	UNION
	SELECT j5.id FROM jobs j5 WHERE j5.knowledge LIKE 'キャビンアテンダント%' AND j5.publish_status = 1 AND (j5.deleted) IS NULL 
	UNION
	SELECT j6.id FROM jobs j6 WHERE j6.location LIKE 'キャビンアテンダント%' AND j6.publish_status = 1 AND (6.deleted) IS NULL 
	UNION
	SELECT j7.id FROM jobs j7 WHERE j7.activity LIKE 'キャビンアテンダント%' AND j7.publish_status = 1 AND (j7.deleted) IS NULL 
	UNION
	SELECT j8.id FROM jobs j8 WHERE j8.salary_statistic_group LIKE 'キャビンアテンダント%' AND j8.publish_status = 1 AND (j8.deleted) IS NULL 
	UNION
	SELECT j9.id FROM jobs j9 WHERE j9.salary_range_remarks LIKE 'キャビンアテンダント%' AND j9.publish_status = 1 AND (j9.deleted) IS NULL 
	UNION
	SELECT j10.id FROM jobs j10 WHERE j10.restriction LIKE 'キャビンアテンダント%' AND j10.publish_status = 1 AND (j10.deleted) IS NULL 
	UNION
	SELECT j11.id FROM jobs j11 WHERE j11.remarks LIKE 'キャビンアテンダント%' AND j11.publish_status = 1 AND (j11.deleted) IS NULL 
)
OR Personalities.name LIKE 'キャビンアテンダント%'
OR PracticalSkills.name LIKE 'キャビンアテンダント%'
OR BasicAbilities.name LIKE 'キャビンアテンダント%'
OR Tools.name LIKE 'キャビンアテンダント%'
OR CareerPaths.name LIKE 'キャビンアテンダント%'
OR RecQualifications.name LIKE 'キャビンアテンダント%'
OR ReqQualifications.name LIKE 'キャビンアテンダント%')
AND Jobs.publish_status = 1
AND (Jobs.deleted) IS NULL
```