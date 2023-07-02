# ROAD-ACCIDENT-IN-THAILAND-DASHBOARD
## Cleaning and SplitTable
- ติดตั้ง library pandas ในชื่อ pd :
```
import pandas as pd
```
<br />

- อ่านไฟล์ CSV และ Set วันเวลาของข้อมูลเป็น 'วันเวลาที่เกิดเหตุ' โดยรวม column 'วันที่เกิดเหตุ','เวลา' :
```
Accident2019 = pd.read_csv('Raw_Data_accident/accident2019.csv',parse_dates = {'วันเวลาที่เกิดเหตุ' : ['วันที่เกิดเหตุ','เวลา']})
Accident2020 = pd.read_csv('Raw_Data_accident/accident2020.csv',parse_dates = {'วันเวลาที่เกิดเหตุ' : ['วันที่เกิดเหตุ','เวลา']})
Accident2021 = pd.read_csv('Raw_Data_accident/accident2021.csv',parse_dates = {'วันเวลาที่เกิดเหตุ' : ['วันที่เกิดเหตุ','เวลา']})
Accident2022 = pd.read_csv('Raw_Data_accident/accident2022.csv',parse_dates = {'วันเวลาที่เกิดเหตุ' : ['วันที่เกิดเหตุ','เวลา']})
```
<br />

- รวมชุดข้อมูลของทั้ง 4 ปี เป็นชุดเดียวกัน ```ignore_index``` คือการ reset index ใหม่ :
```
Accident2019TO2022 = pd.concat([Accident2019,Accident2020,Accident2021,Accident2022],ignore_index = True)
```
<br />

- ลบ column ที่ไม่ได้ใช้งาน :
```
Accident2019TO2022.drop(['ปีที่เกิดเหตุ','วันที่รายงาน','เวลาที่รายงาน','ACC_CODE','รหัสสายทาง','จำนวนที่เกิดเหตุทั้งหมด (รวมคนเดินเท้า)',
'รถจักรยานยนต์','รถสามล้อเครื่อง','รถยนต์นั่งส่วนบุคคล/รถยนต์นั่งสาธารณะ','รถตู้','รถปิคอัพโดยสาร','รถโดยสารมากกว่า 4 ล้อ','รถปิคอัพบรรทุก 4 ล้อ','รถบรรทุก 6 ล้อ','รถบรรทุกมากกว่า 6 ล้อ ไม่เกิน 10 ล้อ','รถบรรทุกมากกว่า 10 ล้อ (รถพ่วง)','รถอีแต๋น','อื่นๆ','คนเดินเท้า','จำนวนผู้บาดเจ็บสาหัส','จำนวนผู้บาดเจ็บเล็กน้อย'],axis = 1 ,inplace = True)
```
<br />

- ลบ row ที่ไม่มีข้อมูลในฟิวด์ และ reset index ใหม่ :
```
Accident2019TO2022.dropna(inplace = True)
Accident2019TO2022.reset_index(inplace = True)
```
<br />

- Set datetime ของข้อมูล :
```
Accident2019TO2022['วันที่เกิดเหตุ'] = pd.to_datetime(Accident2019TO2022['วันเวลาที่เกิดเหตุ']).dt.date
Accident2019TO2022['เวลาที่เกิดเหตุ'] = pd.to_datetime(Accident2019TO2022['วันเวลาที่เกิดเหตุ']).dt.time
```
<br />

- แยก column เพื่อนำไปใช้ในงานต่อในขั้นต่อไป :
```
Datetime = pd.DataFrame(Accident2019TO2022[['วันที่เกิดเหตุ','เวลา']])
AccLocation = pd.DataFrame(Accident2019TO2022[['หน่วยงาน','สายทาง','ก.ม.','จังหวัด','บริเวณที่เกิดเหตุ/ลักษณะทาง','LATITUDE','LONGITUDE']])
AccCause = pd.DataFrame(Accident2019TO2022[['มูลเหตุสันนิษฐาน','ลักษณะการเกิดอุบัติเหตุ','สภาพอากาศ']])
AccEffect = pd.DataFrame(Accident2019TO2022[['จำนวนผู้เสียชีวิต','รวมจำนวนผู้บาดเจ็บ','จำนวนรถที่เกิดเหตุ (รวมคันที่ 1)']])
AccVehicle = pd.DataFrame(Accident2019TO2022[['รถคันที่ 1']])
```
<br />

- บันทึกไฟล์เป็น CSV file :
```
Datetime.to_csv('Datetime.csv',encoding = 'utf8')
AccLocation.to_csv('AccLocation.csv',encoding = 'utf8')
AccCause.to_csv('AccCause.csv',encoding = 'utf8')
AccEffect.to_csv('AccEffect.csv',encoding = 'utf8')
AccVehicle.to_csv('AccVehicle.csv',encoding = 'utf8')
```
## EDA
- เปลี่ยนชื่อ column ให้ทำความเข้าใจได้ง่ายขึ้น :
```
ACD19_22.rename(columns = {"รถคันที่ 1":"ประเภทรถ","จำนวนรถที่เกิดเหตุ (รวมคันที่ 1)":"จำนวนรถที่เกิดเหตุ"},inplace = True)
```
<br />

- ตรวจสอบค่าสถิติพื้นฐานของชุดข้อมูล :
```
ACD19_22.describe()
```
![image](https://github.com/setthawat121/ROAD-ACCIDENT-IN-THAILAND-DASHBOARD/assets/96307668/2c76f0a0-9dbf-4d70-b23a-5094444f10c5)
<br />ค่า SD ของจำนวนรถน้อยกว่าค่า Mean แสดงว่ารถที่เกิดเหตุในละกรณีส่วนมากจะมีจำนวนที่น้อย
<br />ค่า SD ของจำนวนผู้เสียชีวิตมากกว่าค่า Mean แสดงว่าผู้เสียชีวิตในละกรณีส่วนมากจะมีจำนวนที่มาก
<br />ค่า SD ของรวมจำนวนผู้บาดเจ็บมากกว่าค่า Mean แสดงว่าผู้บาดเจ็บในละกรณีส่วนมากจะมีจำนวนที่มาก  
<br />

- ตรวจสอบค่าในฟิวค์ที่แสดงผลเป็นค่าว่าง (Null) :
```
ACD19_22.isnull().sum()
```
![image](https://github.com/setthawat121/ROAD-ACCIDENT-IN-THAILAND-DASHBOARD/assets/96307668/e03a4808-c84d-414c-ac0c-6ef794671c17)
<br />ไม่มีฟิลด์ไหนที่เป็นค่าว่าง (null)
<br />

- แสดงความแตกต่างของข้อมูลในแต่ละ column :
```
ACD19_22.nunique()
```
![image](https://github.com/setthawat121/ROAD-ACCIDENT-IN-THAILAND-DASHBOARD/assets/96307668/ef680f6d-b781-4b09-957e-7e9ff00acf9b)
<br />

- ตรวจสอบค่าความสัมพันธ์ของแต่ละ column , ยิ่งมีค่าเข้าใกล้ 1 มากแค่ไหน ก็ยิ่งมีความสัมพันธ์กันมากเท่านั้น :
```
ACD19_22.corr()
```
![image](https://github.com/setthawat121/ROAD-ACCIDENT-IN-THAILAND-DASHBOARD/assets/96307668/5648cfdd-bf79-4ad5-a9c0-a8217c9e08b8)
<br /> ชุดข้อมูล จำนวนรถที่เกิดอุบัติเหตุ,จำนวนผู้บาดเจ็บ และจำนวนผู้เสียชีวิต มีความสัมพันธ์กัน
<br />

- เรียงลำดับ column ของจำนวนผู้บาดเจ็บ ```ascending=False``` คือ เลือกจากมากไปน้อย :
```
ACD19_22.sort_values(by="รวมจำนวนผู้บาดเจ็บ",ascending=False).head(10)
```
![image](https://github.com/setthawat121/ROAD-ACCIDENT-IN-THAILAND-DASHBOARD/assets/96307668/32b2cff2-2f3f-4e01-9b44-e4521326d902)
<br />

- แสดงจำนวนรวมของข้อมูลโดยจำแนกตามชนิดของรถ :
```
ACD19_22.groupby("ประเภทรถ").sum()
```
![image](https://github.com/setthawat121/ROAD-ACCIDENT-IN-THAILAND-DASHBOARD/assets/96307668/09f7d4c6-8c67-49cf-9136-c975c67da9a3)
<br />พบว่า รถปิคอัพบรรทุก 4 ล้อ,รถยนต์นั่งส่วนบุคคล และรถจักรยานยนต์ มีความเสียหายมากที่สุดตามลำดับ แต่จำนวนของผู้เสียชีวิตของจักรยานยนต์กลับมีจำนวนมากที่สุด พบว่าระดับความปลอดภัยของจักรยานยนต์มีน้อยกว่ายานพาหนะชนิดอื่น ๆ
<br />

- นับจำนวนดูว่าสภาพอากาศแบบไหนมีผลให้เกิดอุบัติเหตุเยอะที่สุด :
```
Accident20192022.groupby(['สภาพอากาศ'])['สภาพอากาศ'].count()
```
![image](https://github.com/setthawat121/ROAD-ACCIDENT-IN-THAILAND-DASHBOARD/assets/96307668/1cb499cf-ce4d-4a88-a5a6-e91714881d15)
<br />
## Dashboard
**URL link Dashboard:** https://app.powerbi.com/view?r=eyJrIjoiZjU2MDA1NjItNDYwOC00MjI5LWFkMzEtMTE3YjdlZmU4OTdlIiwidCI6IjhiMjdiNjQ2LTQ0YTAtNDZlYi05MDNiLWNhNjAyNmFkYjdmYSIsImMiOjEwfQ%3D%3D

Requirements
- ยานพาหนะชนิด เกิดอุบัติเหตุบ่อยที่สุด
- ช่วงเวลาไหนของปีมีจำนวนอุบัติเหตุเยอะที่สุด
- สภาพอากาศไหนส่งผลต่ออัตราการเกิดอุบัติเหตุเยอะที่สุด
- เส้นทางแบบไหนที่มีผลต่อจำนวนการเกิดอุบัติเหตุเยอะที่สุด
- ประชากรในภูมิภาคใดเกิดอุบัติเหตุเยอะที่สุด

![สกรีนช็อต 2023-07-02 210848](https://github.com/setthawat121/ROAD-ACCIDENT-IN-THAILAND-DASHBOARD/assets/96307668/c349acd8-21e4-4451-ab2e-12de175bf1e8)

![สกรีนช็อต 2023-07-02 210946](https://github.com/setthawat121/ROAD-ACCIDENT-IN-THAILAND-DASHBOARD/assets/96307668/4d0c3749-6b61-481b-b38e-6f9826150ec3)

![สกรีนช็อต 2023-07-02 221310](https://github.com/setthawat121/ROAD-ACCIDENT-IN-THAILAND-DASHBOARD/assets/96307668/cd6baf94-1916-49de-9505-36301327aae4)
