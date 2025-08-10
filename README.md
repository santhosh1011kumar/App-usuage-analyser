# App-usuage-analyser
Got it — you want this condensed into a clean, minimal mini-project version while keeping the main functionality.

Here’s the simplified, lightweight App Usage Analyzer with Privacy Insights — minimal code, single file, but still a full working project.


---

 Project Structure (Minimal)

app_usage_analyzer_min/
│
├── app_usage_analyzer.py   # Main script
├── requirements.txt        # Dependencies
└── output/                 # Generated reports & charts


---

 app_usage_analyzer.py (Mini Version)

import pandas as pd
import matplotlib.pyplot as plt
import random, os
from datetime import datetime, timedelta

# ---------------- Config ----------------
NUM_USERS, NUM_DAYS = 5, 7
OUTPUT_DIR = "output"
os.makedirs(OUTPUT_DIR, exist_ok=True)

# ---------------- Data Generation ----------------
apps = ['YouTube','Instagram','WhatsApp','Chrome','Spotify','Facebook','Gmail','TikTok']
privacy_risk = {'Facebook':'High','Instagram':'High','TikTok':'High',
                'WhatsApp':'Medium','Gmail':'Medium','Chrome':'Medium',
                'YouTube':'Low','Spotify':'Low'}
risk_scores = {'Low':1,'Medium':2,'High':3}

data = []
for u in range(1, NUM_USERS+1):
    for d in range(NUM_DAYS):
        date = datetime.today().date() - timedelta(days=d)
        for _ in range(random.randint(3,8)):
            app = random.choice(apps)
            start_hour = random.randint(6,22)
            duration = random.randint(2,30)
            timestamp = datetime.combine(date, datetime.min.time()) + timedelta(hours=start_hour)
            data.append([u, app, duration, timestamp])

df = pd.DataFrame(data, columns=['user_id','app_name','duration','timestamp'])
df['privacy_risk'] = df['app_name'].map(privacy_risk)
df['risk_score'] = df['privacy_risk'].map(risk_scores)
df['date'] = df['timestamp'].dt.date
df['hour'] = df['timestamp'].dt.hour

# ---------------- Analysis ----------------
app_summary = df.groupby('app_name').agg(
    total_duration=('duration','sum'),
    avg_session=('duration','mean'),
    sessions=('duration','count'),
    risk=('privacy_risk','first')
).sort_values('total_duration', ascending=False)

user_summary = df.groupby('user_id').agg(
    total_time=('duration','sum'),
    high_risk_time=('duration', lambda x: df.loc[x.index][df['privacy_risk']=='High']['duration'].sum())
)
user_summary['high_risk_%'] = (user_summary['high_risk_time'] / user_summary['total_time'])*100
user_summary['alert'] = user_summary['high_risk_%'].apply(lambda x: '⚠ High' if x>40 else '✅ Normal')

# ---------------- Visuals ----------------
app_summary.plot(kind='barh', y='total_duration', legend=False, title="Total Duration per App")
plt.xlabel("Minutes"); plt.ylabel("App"); plt.tight_layout()
plt.savefig(f"{OUTPUT_DIR}/app_usage_summary.png"); plt.close()

df.groupby('hour')['duration'].sum().plot(kind='bar', title="Usage by Hour")
plt.xlabel("Hour"); plt.ylabel("Total Minutes"); plt.tight_layout()
plt.savefig(f"{OUTPUT_DIR}/usage_by_hour.png"); plt.close()

# ---------------- Save Reports ----------------
app_summary.to_csv(f"{OUTPUT_DIR}/app_usage_summary.csv")
user_summary.to_csv(f"{OUTPUT_DIR}/user_privacy_report.csv")

# ---------------- Recommendations ----------------
print("\n Privacy Recommendations:")
for uid, row in user_summary.iterrows():
    print(f"User {uid}: {'Reduce high-risk app time!' if row['alert']=='⚠ High' else 'Usage balanced.'}")
print(f"\n Reports saved in '{OUTPUT_DIR}' folder.")


 # Key Reductions from Full Project

Removed Seaborn (just Matplotlib for lighter dependency).

Reduced visualizations to two main charts (App Usage & Usage by Hour).

Condensed data generation loop to ~15 lines.

Shortened analysis & recommendations logic.

CSV outputs kept for reproducibility.



