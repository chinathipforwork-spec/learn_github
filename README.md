# learn_github

import pandas as pd

# โหลดไฟล์
rec26 = pd.read_csv("rec26_2.csv")

# 1. forward fill symbol และข้อมูลที่เกี่ยวข้อง
rec26['ชื่อย่อหลักทรัพย์'] = rec26['ชื่อย่อหลักทรัพย์'].ffill()
rec26['ราคาล่าสุด'] = rec26['ราคาล่าสุด'].ffill()
rec26['Total Coverage'] = rec26['Total Coverage'].ffill()
rec26['Median *'] = rec26['Median *'].ffill()
rec26['Average *'] = rec26['Average *'].ffill()

# 2. extract buy/hold/sell
rec26['buy'] = rec26['Analyst Recommendation'].str.extract(r'Buy(\d+)').astype(float)
rec26['hold'] = rec26['Analyst Recommendation'].str.extract(r'Hold(\d+)').astype(float)
rec26['sell'] = rec26['Analyst Recommendation'].str.extract(r'Sell(\d+)').astype(float)

# 3. เติม NaN ด้วย 0
rec26[['buy','hold','sell']] = rec26[['buy','hold','sell']].fillna(0).astype(int)

# 4. group by symbol แล้ว sum buy/hold/sell
df_group = rec26.groupby('ชื่อย่อหลักทรัพย์', as_index=False).agg({
    'ราคาล่าสุด':'first',
    'Total Coverage':'first',
    'buy':'sum',
    'hold':'sum',
    'sell':'sum',
    'Median *':'first',
    'Average *':'first'
})

# 5. รีเนมคอลัมน์
df_final_rec = df_group.rename(columns={
    'ชื่อย่อหลักทรัพย์':'symbol',
    'ราคาล่าสุด':'entry_price',
    'Total Coverage':'cover',
    'Median *':'median',
    'Average *':'target_profit'
})

print(df_final_rec.head())
