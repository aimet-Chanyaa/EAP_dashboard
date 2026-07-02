---
title: EAP Mental Wellness Dashboard
---

📅 **Q2 2026 (Apr–Jun)** · 👥 **Total Employees: 3,000** · 🔒 **Anonymized · Org-level**

```sql account_users
SELECT COUNT(DISTINCT user_id) AS total
FROM emmind_data.emmind_synthesis_data
```

```sql active_users
SELECT COUNT(DISTINCT user_id) AS total
FROM emmind_data.emmind_synthesis_data
WHERE event_id IS NOT NULL
```

```sql blocked_users
SELECT
  SUM(CASE WHEN blocked = 1 THEN 1 ELSE 0 END) AS blocked_count,
  COUNT(DISTINCT user_id) AS total_accounts,
  ROUND(100.0 * SUM(CASE WHEN blocked = 1 THEN 1 ELSE 0 END) / COUNT(DISTINCT user_id), 1) AS block_rate
FROM (
  SELECT DISTINCT user_id, blocked
  FROM emmind_data.emmind_synthesis_data
)
```

```sql adoption_rate
SELECT
  ROUND(100.0 * COUNT(DISTINCT CASE WHEN event_id IS NOT NULL THEN user_id END) / 3000, 0) AS pct
FROM emmind_data.emmind_synthesis_data
```

<div id="s-reach"></div>

## Reach & Adoption

How employees access Emmind — new friends added, reachable users, and block rate.

<div class="kpis">

<div class="kpi">

🤝 <BigValue
  data={account_users}
  value=total
  title="Account Users"
  fmt="num0"
/>

</div>

<div class="kpi">

✅ <BigValue
  data={active_users}
  value=total
  title="Active Users"
  fmt="num0"
/>

</div>

<div class="kpi k-red">

🚫 <BigValue
  data={blocked_users}
  value=blocked_count
  title="Blocked"
  fmt="num0"
/>

</div>

<div class="kpi">

📣 <BigValue
  data={adoption_rate}
  value=pct
  title="Adoption Rate"
  fmt="num0'%'"
/>

</div>

</div>

<Grid cols=2>

<div>

### Acquisition Channel

How new friends found Emmind — used to plan the next outreach push.

```sql acquisition_path
SELECT
  acquisition_channel,
  COUNT(DISTINCT user_id) AS users,
  ROUND(100.0 * COUNT(DISTINCT user_id) / SUM(COUNT(DISTINCT user_id)) OVER (), 0) AS pct
FROM emmind_data.emmind_synthesis_data
GROUP BY acquisition_channel
ORDER BY users DESC
```

<BarChart
  data={acquisition_path}
  x=acquisition_channel
  y=users
  swapXY=true
  labels=true
  height=220
/>

<DataTable data={acquisition_path} rows=4>
  <Column id=acquisition_channel title="Channel" />
  <Column id=users title="Users" fmt="num0" />
  <Column id=pct title="% of Total" fmt="num0'%'" />
</DataTable>

</div>

<div>

### Monthly Signups & Blocks

New friends per month vs. blocks — a satisfaction signal.

```sql monthly_trend
SELECT
  DATE_TRUNC('month', CAST(signup_date AS DATE)) AS month,
  COUNT(DISTINCT user_id) AS new_signups,
  SUM(CASE WHEN blocked = 1 THEN 1 ELSE 0 END) AS blocked
FROM (
  SELECT DISTINCT user_id, signup_date, blocked
  FROM emmind_data.emmind_synthesis_data
)
GROUP BY month
ORDER BY month
```

<LineChart
  data={monthly_trend}
  x=month
  y=new_signups
  y2=blocked
  yAxisTitle="New Signups"
  y2AxisTitle="Blocked"
  height=240
/>

</div>

</Grid>

<div id="s-demo"></div>

## Demographics

Who uses Emmind — used to design content and activities that fit each group.

<Grid cols=2>

<div>

### Age Band

```sql age_demo
SELECT
  age_band,
  COUNT(DISTINCT user_id) AS users,
  ROUND(100.0 * COUNT(DISTINCT user_id) / SUM(COUNT(DISTINCT user_id)) OVER (), 0) AS pct
FROM (
  SELECT DISTINCT user_id, age_band
  FROM emmind_data.emmind_synthesis_data
)
GROUP BY age_band
ORDER BY age_band
```

<BarChart
  data={age_demo}
  x=age_band
  y=pct
  yAxisTitle="% of Users"
  height=215
/>

</div>

<div>

### Gender

```sql gender_demo
SELECT
  gender,
  COUNT(DISTINCT user_id) AS users,
  ROUND(100.0 * COUNT(DISTINCT user_id) / SUM(COUNT(DISTINCT user_id)) OVER (), 0) AS pct
FROM (
  SELECT DISTINCT user_id, gender
  FROM emmind_data.emmind_synthesis_data
)
GROUP BY gender
ORDER BY users DESC
```

<BarChart
  data={gender_demo}
  x=gender
  y=users
  swapXY=true
  labels=true
  height=215
/>

</div>

</Grid>

<div id="s-eng"></div>

## Engagement

What employees do once they're in Emmind — which features get used, how deeply, and how far they get through guided skill paths.

<Grid cols=2>

<div>

### Feature Usage

Which modules get opened most — used to prioritize what to promote next.

```sql module_usage
SELECT
  module,
  COUNT(*) AS events
FROM emmind_data.emmind_synthesis_data
WHERE module IS NOT NULL AND module != ''
GROUP BY module
ORDER BY events DESC
```

<BarChart
  data={module_usage}
  x=module
  y=events
  swapXY=true
  labels=true
  height=240
/>

</div>

<div>

### Engagement Tier

Share of users classed as High / Mid / Low engagement.

```sql engagement_tier
SELECT
  engagement_tier,
  COUNT(DISTINCT user_id) AS users,
  ROUND(100.0 * COUNT(DISTINCT user_id) / SUM(COUNT(DISTINCT user_id)) OVER (), 0) AS pct
FROM emmind_data.emmind_synthesis_data
GROUP BY engagement_tier
ORDER BY users DESC
```

<BarChart
  data={engagement_tier}
  x=engagement_tier
  y=users
  swapXY=true
  labels=true
  height=240
/>

</div>

</Grid>

### Skill Path Progress — Started vs. Completed

Completion rate of each guided CBT-style skill path — shows where to nudge users who drop off.

```sql skill_path_progress
SELECT
  path_name,
  COUNT(DISTINCT user_id) AS started,
  COUNT(DISTINCT CASE WHEN skill_completed = 1 THEN user_id END) AS completed
FROM emmind_data.emmind_synthesis_data
WHERE path_name IS NOT NULL AND path_name != ''
GROUP BY path_name
ORDER BY started DESC
```

<BarChart
  data={skill_path_progress}
  x=path_name
  y={['started','completed']}
  yAxisTitle="Users"
  height=280
/>

<div id="s-health"></div>

## Wellbeing — 9Q Screening

Depression screening results (9Q, Department of Mental Health criteria) from actual users who completed the test, plus what they log about their mood.

```sql q9_severity
SELECT
  CASE
    WHEN q9_score < 7 THEN '1 · None/Minimal (0–6)'
    WHEN q9_score BETWEEN 7 AND 12 THEN '2 · Mild (7–12)'
    WHEN q9_score BETWEEN 13 AND 18 THEN '3 · Moderate (13–18)'
    WHEN q9_score >= 19 THEN '4 · Severe (19–27)'
  END AS severity_band,
  COUNT(DISTINCT user_id) AS users
FROM emmind_data.emmind_synthesis_data
WHERE q9_score IS NOT NULL AND q9_score != ''
GROUP BY severity_band
ORDER BY severity_band
```

<BarChart
  data={q9_severity}
  x=severity_band
  y=users
  swapXY=true
  labels=true
  height=240
/>

```sql safety_funnel
SELECT
  SUM(CASE WHEN high_severity_flag = 1 THEN 1 ELSE 0 END) AS step1_high_severity,
  SUM(safety_plan_completed) AS step2_safety_plan_completed,
  SUM(sos_clicked) AS step3_sos_clicked
FROM emmind_data.emmind_synthesis_data
```

### Safety Funnel

<div class="kpis">

<div class="kpi k-red">

🆘 <BigValue data={safety_funnel} value=step1_high_severity title="High Severity (9Q ≥ 19)" fmt="num0" />

</div>

<div class="kpi">

🛟 <BigValue data={safety_funnel} value=step2_safety_plan_completed title="Safety Plan Completed" fmt="num0" />

</div>

<div class="kpi k-red">

📞 <BigValue data={safety_funnel} value=step3_sos_clicked title="SOS Help Clicked" fmt="num0" />

</div>

</div>

### Trend — Share Flagged High Severity

Monthly % of 9Q test-takers scoring in the high-severity range — an early-warning trend, not a one-off snapshot.

```sql q9_trend
SELECT
  DATE_TRUNC('month', CAST(event_date AS DATE)) AS month,
  ROUND(100.0 * SUM(CASE WHEN high_severity_flag = 1 THEN 1 ELSE 0 END) / COUNT(*), 1) AS high_severity_pct
FROM emmind_data.emmind_synthesis_data
WHERE q9_score IS NOT NULL AND q9_score != ''
GROUP BY month
ORDER BY month
```

<LineChart
  data={q9_trend}
  x=month
  y=high_severity_pct
  yAxisTitle="% High Severity"
  height=240
/>

<Grid cols=2>

<div>

### Mood Logged

```sql mood_breakdown
SELECT
  mood_logged,
  COUNT(*) AS log_count
FROM emmind_data.emmind_synthesis_data
WHERE mood_logged IS NOT NULL AND mood_logged != ''
GROUP BY mood_logged
ORDER BY log_count DESC
```

<BarChart
  data={mood_breakdown}
  x=mood_logged
  y=log_count
  swapXY=true
  labels=true
  height=220
/>

</div>

<div>

### Mood Factors

When a negative mood is logged, what employees say is driving it — points to root causes HR can actually act on.

```sql mood_factor
SELECT
  mood_factor,
  COUNT(*) AS log_count
FROM emmind_data.emmind_synthesis_data
WHERE mood_factor IS NOT NULL AND mood_factor != ''
GROUP BY mood_factor
ORDER BY log_count DESC
```

<BarChart
  data={mood_factor}
  x=mood_factor
  y=log_count
  swapXY=true
  labels=true
  height=220
/>

</div>

</Grid>

<div id="s-insight"></div>

## Deep Insight — Department-Level Patterns

Cutting wellbeing data by department surfaces where to focus support, rather than treating the org as one flat number.

```sql dept_severity
SELECT
  department,
  COUNT(DISTINCT CASE WHEN q9_score IS NOT NULL AND q9_score != '' THEN user_id END) AS q9_takers,
  COUNT(DISTINCT CASE WHEN high_severity_flag = 1 THEN user_id END) AS high_severity_users,
  ROUND(
    100.0 * COUNT(DISTINCT CASE WHEN high_severity_flag = 1 THEN user_id END)
    / NULLIF(COUNT(DISTINCT CASE WHEN q9_score IS NOT NULL AND q9_score != '' THEN user_id END), 0)
  , 1) AS high_severity_pct
FROM emmind_data.emmind_synthesis_data
GROUP BY department
ORDER BY high_severity_pct DESC
```

### High-Severity 9Q Rate by Department

<BarChart
  data={dept_severity}
  x=department
  y=high_severity_pct
  swapXY=true
  labels=true
  yAxisTitle="% High Severity"
  height=280
/>

```sql dept_top_factor
SELECT department, mood_factor, log_count
FROM (
  SELECT
    department,
    mood_factor,
    COUNT(*) AS log_count,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY COUNT(*) DESC) AS rn
  FROM emmind_data.emmind_synthesis_data
  WHERE mood_factor IS NOT NULL AND mood_factor != ''
  GROUP BY department, mood_factor
)
WHERE rn = 1
ORDER BY log_count DESC
```

### Top Mood Factor by Department

<DataTable data={dept_top_factor} rows=10>
  <Column id=department title="Department" />
  <Column id=mood_factor title="Top Factor" />
  <Column id=log_count title="Mentions" fmt="num0" />
</DataTable>

<div id="s-action"></div>

## Action Plan

Recommendations generated directly from the metrics above — each one cites the number behind it.

```sql action_stats
SELECT
  (SELECT mood_factor FROM emmind_data.emmind_synthesis_data
   WHERE mood_factor IS NOT NULL AND mood_factor != ''
   GROUP BY mood_factor ORDER BY COUNT(*) DESC LIMIT 1) AS top_factor,
  (SELECT department FROM emmind_data.emmind_synthesis_data
   WHERE high_severity_flag = 1
   GROUP BY department ORDER BY COUNT(*) DESC LIMIT 1) AS top_severity_dept
```

<div class="actions">

<div class="action">
<span class="tag t-now">🆘 Act Now</span>
<h4>Support the high-severity group proactively</h4>
<p>Put "SOS Help" and the crisis line front-and-center in the app, and train managers to spot warning signs and refer people.</p>
<div class="why">Reference: <b>{safety_funnel[0].step1_high_severity}</b> users flagged high severity (9Q ≥ 19) · SOS clicked <b>{safety_funnel[0].step3_sos_clicked}</b> times this period · highest-severity department: <b>{action_stats[0].top_severity_dept}</b></div>
</div>

<div class="action">
<span class="tag t-soon">⚖️ Urgent</span>
<h4>Address the #1 root cause: {action_stats[0].top_factor}</h4>
<p>Review workload distribution and time-boundary policies, starting with the highest-severity department identified above.</p>
<div class="why">Reference: top mood factor is <b>{action_stats[0].top_factor}</b> · department with most high-severity flags: <b>{action_stats[0].top_severity_dept}</b></div>
</div>

<div class="action">
<span class="tag t-plan">📣 Plan</span>
<h4>Cut blocks & push adoption toward target</h4>
<p>Send useful (non-promotional) broadcasts and re-engage users who have gone quiet, rather than broad blasts.</p>
<div class="why">Reference: adoption <b>{adoption_rate[0].pct}%</b> of employees · block rate <b>{blocked_users[0].block_rate}%</b></div>
</div>

</div>

<LastRefreshed prefix="Data last refreshed" />
