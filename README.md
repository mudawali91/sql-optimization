## SQL Optimization

### Issue

There are a `SQL` query that produces a search result. The query produces results in approximately 8 seconds.

### Purpose
To suggest improvements that can be done to the query in order to improve its
performance.

---

### Total Hour Spent
+- 4 Hours

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

**6.Minimize number of tables `JOIN`**
* Reduce multiple tables joined that is not related 
> **Note:**: In case of the query, the table `affiliates` has joined multiple time just only for filtering and not select any column for display as well as its relation tables `jobs_tools`, `jobs_career_paths`, `jobs_rec_qualifications` and `jobs_req_qualifications`.
> Table `affiliates` also has multiple `OR` usage to filter the column `name` where they actually came from the same table.
> So that we can **replace** all the `LEFT JOIN` came from these tables and try to use `INNER JOIN` with subquery because it will result from both table to match.
```
INNER JOIN jobs_tools JobsTools
ON ( Jobs.id = (JobsTools.job_id)
AND JobsTools.affiliate_id IN (SELECT Tools.id FROM affiliates Tools WHERE Tools.type = 1 AND (Tools.deleted IS NULL) AND Tools.name LIKE 'キャビンアテンダント%') )

INNER JOIN jobs_career_paths JobsCareerPaths
ON ( Jobs.id = (JobsCareerPaths.job_id)
AND JobsCareerPaths.affiliate_id IN (SELECT CareerPaths.id FROM affiliates CareerPaths WHERE CareerPaths.type = 3 AND (CareerPaths.deleted IS NULL) AND CareerPaths.name LIKE 'キャビンアテンダント%') )

INNER JOIN jobs_rec_qualifications JobsRecQualifications
ON ( Jobs.id = (JobsRecQualifications.job_id)
AND JobsRecQualifications.affiliate_id IN (SELECT RecQualifications.id FROM affiliates RecQualifications WHERE RecQualifications.type = 2 AND (RecQualifications.deleted IS NULL) AND RecQualifications.name LIKE 'キャビンアテンダント%') )

INNER JOIN jobs_req_qualifications JobsReqQualifications
ON ( Jobs.id = (JobsReqQualifications.job_id)
AND JobsReqQualifications.affiliate_id IN (SELECT ReqQualifications.id FROM affiliates ReqQualifications WHERE ReqQualifications.type = 2 AND (ReqQualifications.deleted IS NULL) AND ReqQualifications.name LIKE 'キャビンアテンダント%') )
```
> AND also for table `jobs_personalities` with `personalities`, `jobs_practical_skills` with `practical_skills` and `jobs_basic_abilities` with `basic_abilities`, where those tables only joined for filtering purpose. So we can use `INNER JOIN` with subquery.
```
INNER JOIN jobs_personalities JobsPersonalities
ON ( Jobs.id = (JobsPersonalities.job_id)
AND JobsPersonalities.personality_id IN (SELECT Personalities.id FROM personalities Personalities WHERE (Personalities.deleted) IS NULL AND Personalities.name LIKE 'キャビンアテンダント%') )

INNER JOIN jobs_practical_skills JobsPracticalSkills
ON ( Jobs.id = (JobsPracticalSkills.job_id)
AND JobsPracticalSkills.practical_skill_id IN (SELECT PracticalSkills.id FROM practical_skills PracticalSkills WHERE (PracticalSkills.deleted) IS NULL AND PracticalSkills.name LIKE '%キャビンアテンダント%') )

INNER JOIN jobs_basic_abilities JobsBasicAbilities
ON ( Jobs.id = (JobsBasicAbilities.job_id)
AND JobsBasicAbilities.basic_ability_id IN (SELECT BasicAbilities.id FROM basic_abilities BasicAbilities WHERE (BasicAbilities.deleted) IS NULL AND BasicAbilities.name LIKE '%キャビンアテンダント%') )
```

> Remove the `OR` for all related to table `affiliate` in `WHERE` clause as we already use the filter in subquery above
```
~~OR Personalities.name LIKE '%キャビンアテンダント%'~~
~~OR PracticalSkills.name LIKE '%キャビンアテンダント%'~~
~~OR BasicAbilities.name LIKE '%キャビンアテンダント%'~~
~~OR Tools.name LIKE 'キャビンアテンダント%'~~
~~OR CareerPaths.name LIKE 'キャビンアテンダント%'~~
~~OR RecQualifications.name LIKE 'キャビンアテンダント%'~~
~~OR ReqQualifications.name LIKE 'キャビンアテンダント%'~~
```

---

###Final Query
```
SELECT Jobs.id AS `Jobs__id`,
Jobs.name AS `Jobs__name`,
Jobs.media_id AS `Jobs__media_id`,
Jobs.job_category_id AS `Jobs__job_category_id`,
Jobs.job_type_id AS `Jobs__job_type_id`,
Jobs.description AS `Jobs__description`,
Jobs.detail AS `Jobs__detail`,
Jobs.business_skill AS `Jobs__business_skill`,
Jobs.knowledge AS `Jobs__knowledge`,
Jobs.location AS `Jobs__location`,
Jobs.activity AS `Jobs__activity`,
Jobs.academic_degree_doctor AS `Jobs__academic_degree_doctor`,
Jobs.academic_degree_master AS `Jobs__academic_degree_master`,
Jobs.academic_degree_professional AS `Jobs__academic_degree_professional`,
Jobs.academic_degree_bachelor AS `Jobs__academic_degree_bachelor`,
Jobs.salary_statistic_group AS `Jobs__salary_statistic_group`,
Jobs.salary_range_first_year AS `Jobs__salary_range_first_year`,
Jobs.salary_range_average AS `Jobs__salary_range_average`,
Jobs.salary_range_remarks AS `Jobs__salary_range_remarks`,
Jobs.restriction AS `Jobs__restriction`,
Jobs.estimated_total_workers AS `Jobs__estimated_total_workers`,
Jobs.remarks AS `Jobs__remarks`,
Jobs.url AS `Jobs__url`,
Jobs.publish_status AS `Jobs__publish_status`,
Jobs.version AS `Jobs__version`,
JobCategories.id AS `JobCategories__id`,
JobCategories.name AS `JobCategories__name`,
JobTypes.id AS `JobTypes__id`,
JobTypes.name AS `JobTypes__name`,
JobTypes.job_category_id AS `JobTypes__job_category_id`,

FROM jobs Jobs

INNER JOIN jobs_personalities JobsPersonalities
ON ( Jobs.id = (JobsPersonalities.job_id)
AND JobsPersonalities.personality_id IN (SELECT Personalities.id FROM personalities Personalities WHERE (Personalities.deleted) IS NULL AND Personalities.name LIKE 'キャビンアテンダント%') )

INNER JOIN jobs_practical_skills JobsPracticalSkills
ON ( Jobs.id = (JobsPracticalSkills.job_id)
AND JobsPracticalSkills.practical_skill_id IN (SELECT PracticalSkills.id FROM practical_skills PracticalSkills WHERE (PracticalSkills.deleted) IS NULL AND PracticalSkills.name LIKE '%キャビンアテンダント%') )

INNER JOIN jobs_basic_abilities JobsBasicAbilities
ON ( Jobs.id = (JobsBasicAbilities.job_id)
AND JobsBasicAbilities.basic_ability_id IN (SELECT BasicAbilities.id FROM basic_abilities BasicAbilities WHERE (BasicAbilities.deleted) IS NULL AND BasicAbilities.name LIKE '%キャビンアテンダント%') )

INNER JOIN jobs_tools JobsTools
ON ( Jobs.id = (JobsTools.job_id)
AND JobsTools.affiliate_id IN (SELECT Tools.id FROM affiliates Tools WHERE Tools.type = 1 AND (Tools.deleted IS NULL) AND Tools.name LIKE 'キャビンアテンダント%') )

INNER JOIN jobs_career_paths JobsCareerPaths
ON ( Jobs.id = (JobsCareerPaths.job_id)
AND JobsCareerPaths.affiliate_id IN (SELECT CareerPaths.id FROM affiliates CareerPaths WHERE CareerPaths.type = 3 AND (CareerPaths.deleted IS NULL) AND CareerPaths.name LIKE 'キャビンアテンダント%') )

INNER JOIN jobs_rec_qualifications JobsRecQualifications
ON ( Jobs.id = (JobsRecQualifications.job_id)
AND JobsRecQualifications.affiliate_id IN (SELECT RecQualifications.id FROM affiliates RecQualifications WHERE RecQualifications.type = 2 AND (RecQualifications.deleted IS NULL) AND RecQualifications.name LIKE 'キャビンアテンダント%') )

INNER JOIN jobs_req_qualifications JobsReqQualifications
ON ( Jobs.id = (JobsReqQualifications.job_id)
AND JobsReqQualifications.affiliate_id IN (SELECT ReqQualifications.id FROM affiliates ReqQualifications WHERE ReqQualifications.type = 2 AND (ReqQualifications.deleted IS NULL) AND ReqQualifications.name LIKE 'キャビンアテンダント%') )

INNER JOIN job_categories JobCategories
ON (JobCategories.id = (Jobs.job_category_id)
AND (JobCategories.deleted) IS NULL)

INNER JOIN job_types JobTypes
ON (JobTypes.id = (Jobs.job_type_id)
AND (JobTypes.deleted) IS NULL)

WHERE ((JobCategories.name LIKE '%キャビンアテンダント%'
OR JobTypes.name LIKE '%キャビンアテンダント%'
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
))
AND Jobs.publish_status = 1
AND (Jobs.deleted) IS NULL)
GROUP BY Jobs.id
ORDER BY Jobs.sort_order desc,
Jobs.id DESC LIMIT 50 OFFSET 0
```