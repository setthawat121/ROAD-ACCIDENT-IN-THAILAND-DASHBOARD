# ROAD-ACCIDENT-IN-THAILAND-DASHBOARD
## Cleaning and SplitTable
ติดตั้ง library pandas ในชื่อ pd
```
import pandas as pd
```
อ่านไฟล์ CSV และ Set วันเวลาของข้อมูลเป็น 'วันเวลาที่เกิดเหตุ' โดยรวม column 'วันที่เกิดเหตุ','เวลา'
```
Accident2019 = pd.read_csv('accident2019.csv',parse_dates = {'วันเวลาที่เกิดเหตุ' : ['วันที่เกิดเหตุ','เวลา']})
Accident2020 = pd.read_csv('accident2020.csv',parse_dates = {'วันเวลาที่เกิดเหตุ' : ['วันที่เกิดเหตุ','เวลา']})
Accident2021 = pd.read_csv('accident2021.csv',parse_dates = {'วันเวลาที่เกิดเหตุ' : ['วันที่เกิดเหตุ','เวลา']})
Accident2022 = pd.read_csv('accident2022.csv',parse_dates = {'วันเวลาที่เกิดเหตุ' : ['วันที่เกิดเหตุ','เวลา']})
```
รวมชุดข้อมูลของทั้ง 4 ปี เป็นชุดเดียวกัน ```ignore_index``` คือการ reset index ใหม่
```
Accident2019TO2022 = pd.concat([Accident2019,Accident2020,Accident2021,Accident2022],ignore_index = True)
```
ลบ column ที่ไม่ได้ใช้งาน
```
Accident2019TO2022.drop(['ปีที่เกิดเหตุ','วันที่รายงาน','เวลาที่รายงาน','ACC_CODE','รหัสสายทาง','จำนวนที่เกิดเหตุทั้งหมด (รวมคนเดินเท้า)',
'รถจักรยานยนต์','รถสามล้อเครื่อง','รถยนต์นั่งส่วนบุคคล/รถยนต์นั่งสาธารณะ','รถตู้','รถปิคอัพโดยสาร','รถโดยสารมากกว่า 4 ล้อ','รถปิคอัพบรรทุก 4 ล้อ','รถบรรทุก 6 ล้อ','รถบรรทุกมากกว่า 6 ล้อ ไม่เกิน 10 ล้อ','รถบรรทุกมากกว่า 10 ล้อ (รถพ่วง)','รถอีแต๋น','อื่นๆ','คนเดินเท้า','จำนวนผู้บาดเจ็บสาหัส','จำนวนผู้บาดเจ็บเล็กน้อย'],axis = 1 ,inplace = True)
```
ลบ row ที่ไม่มีข้อมูลในฟิวด์ และ reset index ใหม่
```
Accident2019TO2022.dropna(inplace = True)
Accident2019TO2022.reset_index(inplace = True)
```
Set datetime ของข้อมูล
```
Accident2019TO2022['วันที่เกิดเหตุ'] = pd.to_datetime(Accident2019TO2022['วันเวลาที่เกิดเหตุ']).dt.date
Accident2019TO2022['เวลาที่เกิดเหตุ'] = pd.to_datetime(Accident2019TO2022['วันเวลาที่เกิดเหตุ']).dt.time
```
แยก column เพื่อนำไปใช้ในงานต่อในขั้นต่อไป
```
Datetime = pd.DataFrame(Accident2019TO2022[['วันที่เกิดเหตุ','เวลา']])
AccLocation = pd.DataFrame(Accident2019TO2022[['หน่วยงาน','สายทาง','ก.ม.','จังหวัด','บริเวณที่เกิดเหตุ/ลักษณะทาง','LATITUDE','LONGITUDE']])
AccCause = pd.DataFrame(Accident2019TO2022[['มูลเหตุสันนิษฐาน','ลักษณะการเกิดอุบัติเหตุ','สภาพอากาศ']])
AccEffect = pd.DataFrame(Accident2019TO2022[['จำนวนผู้เสียชีวิต','รวมจำนวนผู้บาดเจ็บ','จำนวนรถที่เกิดเหตุ (รวมคันที่ 1)']])
AccVehicle = pd.DataFrame(Accident2019TO2022[['รถคันที่ 1']])
```
บันทึกไฟล์เป็น CSV file
```
Datetime.to_csv('Datetime.csv',encoding = 'utf8')
AccLocation.to_csv('AccLocation.csv',encoding = 'utf8')
AccCause.to_csv('AccCause.csv',encoding = 'utf8')
AccEffect.to_csv('AccEffect.csv',encoding = 'utf8')
AccVehicle.to_csv('AccVehicle.csv',encoding = 'utf8')
```
## EDA
เปลี่ยนชื่อ column ให้ทำความเข้าใจได้ง่ายขึ้น
```
ACD19_22.rename(columns = {"รถคันที่ 1":"ประเภทรถ","จำนวนรถที่เกิดเหตุ (รวมคันที่ 1)":"จำนวนรถที่เกิดเหตุ"},inplace = True)
```
ตรวจสอบค่าสถิติพื้นฐานของชุดข้อมูล
```
ACD19_22.describe()
```
ตรวจสอบค่าในฟิวค์ที่แสดงผลเป็นค่าว่าง (Null)
```
ACD19_22.isnull().sum()
```
แสดงความแตกต่างของข้อมูลในแต่ละ column
```
ACD19_22.nunique()
```
ตรวจสอบค่าความสัมพันธ์ของแต่ละ column

ยิ่งมีค่าเข้าใกล้ 1 มากแค่ไหน ก็ยิ่งมีความสัมพันธ์กันมากเท่านั้น
```
ACD19_22.corr()
```
เรียงลำดับ column ของจำนวนผู้บาดเจ็บ ```ascending=False``` คือ เลือกจากมากไปน้อย
```
ACD19_22.sort_values(by="รวมจำนวนผู้บาดเจ็บ",ascending=False).head(10)
```
แสดงจำนวนรวมของข้อมูลโดยจำแนกตามชนิดของรถ
```
ACD19_22.groupby("ประเภทรถ").sum()
```
นับจำนวนดูว่าสภาพอากาศแบบไหนมีผลให้เกิดอุบัติเหตุเยอะที่สุด
```
Accident20192022.groupby(['สภาพอากาศ'])['สภาพอากาศ'].count()
```
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
