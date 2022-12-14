# Optimal-hemoglobin-concentration-and-the-significance-of-early-transfusion-therapy-in-elder- sepsis-patient
import pandas as pd
ICU_patients = pd.read_csv('D:/hongxiboandsepsis/icustay_detail.csv')
ICU_patients.columns
ICU_patients = ICU_patients[(ICU_patients.hospstay_seq == 1) & (ICU_patients.icustay_seq == 1)]
ICU_patients.sort_values(by=['subject_id'],inplace=True)
ICU_patients.duplicated(subset=['subject_id']).sum()
#导入脓毒症
sepsis3 = pd.read_csv('D:\hongxiboandsepsis\sepsis3.csv')
sepsis3.columns
### 24小时内留取培养 3天内使用抗生素
sepsis3['suspected_infection_time'] = pd.to_datetime(sepsis3['suspected_infection_time'])
sepsis3['antibiotic_time']= pd.to_datetime(sepsis3['antibiotic_time'])
sepsis3['culture_time'] = pd.to_datetime(sepsis3['culture_time'])
sepsis3['delt1'] = (sepsis3.antibiotic_time -
                    sepsis3.suspected_infection_time   ).astype('timedelta64[h]')

sepsis3['delt2'] = (sepsis3.culture_time -
                     sepsis3.suspected_infection_time ).astype('timedelta64[h]')

sepsis3=sepsis3[(sepsis3.delt1 <=72) |(sepsis3.delt2 <=24)]

sepsis3 = sepsis3.drop(labels=['culture_time','antibiotic_time', 'respiration','coagulation', 'liver', 'cardiovascular', 'cns', 
'renal','sepsis3','delt1','delt2'],axis=1)


sepsis3.columns
# 剔除重复数据
sepsis3.duplicated(subset = ['subject_id']).sum()
sepsis3.columns
sepsis3.info()
sepsis3['sepsis'] = 1

##合并sepsis
sepsis_ICU = pd.merge(ICU_patients, sepsis3,how='inner',on = ['subject_id','stay_id'])
sepsis_ICU.columns
#####怀疑感染在住ICU前后24小时
sepsis_ICU['icu_intime'] =  pd.to_datetime(sepsis_ICU['icu_intime'])
sepsis_ICU['timedelat1'] = (sepsis_ICU.suspected_infection_time -
                             sepsis_ICU.icu_intime).astype('timedelta64[h]')
sepsis_ICU['timedelat1'] = sepsis_ICU.timedelat1.astype(int)
sepsis_ICU = sepsis_ICU[(sepsis_ICU.timedelat1 <=24) & (sepsis_ICU.timedelat1 >=-24)]
sepsis_ICU = sepsis_ICU.drop(labels= ['timedelat1'],axis =1)
#### sofa 评分在入ICU24h内
sepsis_ICU['sofa_time'] = pd.to_datetime(sepsis_ICU['sofa_time'] )
sepsis_ICU['timedelat2']=(sepsis_ICU['sofa_time'] -
                             sepsis_ICU['icu_intime']).astype('timedelta64[h]')
sepsis_ICU = sepsis_ICU[(sepsis_ICU.timedelat2 <=24) & (sepsis_ICU.timedelat2 >=0)]
sepsis_ICU = sepsis_ICU.drop(labels= ['timedelat2','sofa_time'],axis =1)

### 读取血气信息
gas = pd.read_csv('D:\hongxiboandsepsis\gas.csv')
sepsis_ICU = pd.merge(sepsis_ICU,gas,on=['subject_id','stay_id'],how='left')
sepsis_ICU.columns
sepsis_ICU.isna().sum()
sepsis_ICU.dropna(axis = 0,subset=['lactate_max'])
sepsis_ICU=sepsis_ICU.dropna(axis = 0,subset=['lactate_max'])
####删除年龄小于65及住ICU>48小时
sepsis_ICU.columns
sepsis_ICU = sepsis_ICU.drop(labels=['ethnicity', 'hospstay_seq',
 'first_hosp_stay','icustay_seq','first_icu_stay'],axis=1)

sepsis_ICU['icu_outtime'] =  pd.to_datetime(sepsis_ICU['icu_outtime'])
sepsis_ICU['los_icu'] =(sepsis_ICU.icu_outtime - 
                        sepsis_ICU.icu_intime).astype('timedelta64[h]')

sepsis_ICU = sepsis_ICU[(sepsis_ICU.admission_age >= 65)&(sepsis_ICU.los_icu>24)]
sepsis_ICU.columns 

# sepsis_ICU.columns =['subject_id', 'hadm_id', 'stay_id', 'gender', 'dod', 'admittime',
#       'dischtime', 'los_hospital', 'admission_age', 'hospital_expire_flag',
#       'icu_intime', 'icu_outtime', 'los_icu', 'suspected_infection_time',
#       'sofa_score', 'sepsis']

sepsis_ICU.columns =['subject_id', 'hadm_id', 'stay_id', 'gender', 'dod', 'admittime',
       'dischtime', 'los_hospital', 'admission_age', 'hospital_expire_flag',
       'icu_intime', 'icu_outtime', 'los_icu', 'suspected_infection_time',
       'sofa_score', 'sepsis', 'lactate', 'ph', 'po2', 'pco2']

### 读取实验室检验
lab = pd.read_csv('D:\hongxiboandsepsis\lab.csv')
lab.columns
##更改列名
lab.columns = ['subject_id', 'stay_id', 'hematocrit', 'hemoglobin','hemoglobin_max',
'platelets', 'wbc', 'albumin', 'bun', 'creatinine','glucose', 'sodium',
 'potassium', 'd_dimer','fibrinogen', 'pt', 'ptt', 'alt', 'ast','bilirubin']

### 合并sepsis和实验室检查表格
sepsis_ICU = pd.merge(sepsis_ICU, lab, on= ['subject_id','stay_id'],how='left')
sepsis_ICU.isna().sum()

### 删除缺失＞10%变量
sepsis_ICU.columns
sepsis_ICU = sepsis_ICU.drop(labels=[ 'albumin', 'd_dimer', 'fibrinogen', 
'alt', 'ast'], axis=1)
sepsis_ICU.isna().sum()

### 导入24小时内血红蛋白平均值
mean_hb = pd.read_csv('D:/hongxiboandsepsis/firsthemoglobin.csv')
mean_hb.columns
mean_hb = mean_hb.drop(labels = ['labevent_id','specimen_id', 'itemid','valuenum', 
'valueuom',  'ref_range_lower', 'ref_range_upper', 'flag', 'priority', 
'comments' ],axis = 1)
sepsis_ICUhb = sepsis_ICU.copy()
sepsis_ICUhb = pd.merge(sepsis_ICU, mean_hb,on=['subject_id'],how='left')
sepsis_ICUhb.columns
sepsis_ICUhb['charttime'] =  pd.to_datetime(sepsis_ICUhb['charttime'])
## 计算血色素测量时间和入ICU时间差
sepsis_ICUhb['timedelat1'] = (sepsis_ICUhb.charttime -sepsis_ICUhb.icu_intime).astype('timedelta64[h]')
sepsis_ICUhb['timedelat1'] = sepsis_ICUhb.timedelat1.astype(float)
sepsis_ICUhb.head(5)
### 仅包括入ICU24小时内血色素
sepsis_ICUhb = sepsis_ICUhb[(sepsis_ICUhb.timedelat1 <= 24)&
                        (sepsis_ICUhb.timedelat1 >=-6)]
### 保留平均值
hb_mean = sepsis_ICUhb.groupby('subject_id',as_index = False)['value'].mean()
hb_mean.columns =['subject_id', 'mean_hb']
hb_mean.dropna(subset=['mean_hb'],inplace=True)
sepsis_ICU = pd.merge(sepsis_ICU, hb_mean,on=['subject_id'],how = 'left')
sepsis_ICU.columns

# 删除第红蛋白平均值是缺失值的变量
sepsis_ICU.dropna(subset=['mean_hb'],inplace=True)
sepsis_ICU.isna().sum()
sepsis_ICU.columns 
sepsis_ICU.columns = ['subject_id', 'hadm_id', 'stay_id', 'gender', 'dod', 'admittime',
       'dischtime', 'los_hospital', 'admission_age', 'hospital_expire_flag',
       'icu_intime', 'icu_outtime', 'los_icu', 'suspected_infection_time',
       'sofa_score', 'sepsis', 'lactate', 'ph', 'po2', 'pco2', 'hematocrit',
       'hemoglobin', 'hemoglobin_max', 'platelets', 'wbc', 'bun', 'creatinine',
       'glucose', 'sodium', 'potassium', 'pt', 'ptt', 'bilirubin', 'mean_hb']

### 导入生命体征
vitalsign = pd.read_csv('D:\hongxiboandsepsis\italsign.csv')
vitalsign.columns
vitalsign.columns = ['subject_id', 'stay_id', 'HR', 'SBP', 'DBP',
       'MBP', 'RR', 'T_min', 'T_max', 'spo2']
###合并生命体征
sepsis_ICU = pd.merge(sepsis_ICU,vitalsign,on=['subject_id', 'stay_id'],how='left')

### 导入体重
weight = pd.read_csv('D:\hongxiboandsepsis\weight.csv')
###合并体重
sepsis_ICU = pd.merge(sepsis_ICU,weight,on=['subject_id', 'stay_id'],how='left')


### 导入身高
high = pd.read_csv('D:\hongxiboandsepsis\height.csv')
###  合并身高
sepsis_ICU = pd.merge(sepsis_ICU, high,on=['subject_id', 'stay_id'],how= 'left')
sepsis_ICU.isna().sum()
###  apsiii 合并
asps = pd.read_csv('D:\hongxiboandsepsis\sapsiii.csv')
asps.columns
asps = asps[['subject_id', 'hadm_id', 'stay_id', 'apsiii']]
sepsis_ICU = pd.merge(sepsis_ICU,asps,on = ['subject_id', 'hadm_id', 'stay_id'],
how = 'left')

### 合并AKI
akipatients = pd.read_csv('D:\hongxiboandsepsis\AKI.csv')
akipatients.columns
### 建立排序
akipatients['aki_rank']= akipatients.groupby('stay_id')['aki_stage_creat'].rank(method='first',
na_option = 'bottom',ascending=False)
### 剔除最大值之外的值
akipatients = akipatients.sort_values(by=['stay_id','aki_rank'])
akipatients.drop_duplicates(subset=['stay_id'],inplace = True)
akipatients = akipatients.drop(labels = ['aki_rank'],axis = 1)
### 合并表格
sepsis_ICU = pd.merge(sepsis_ICU,akipatients,on = ['subject_id', 'hadm_id',
'stay_id'],how= 'left')
sepsis_ICU.columns
sepsis_ICU.loc[sepsis_ICU['aki_stage_creat'] >= 1,'aki_stage_creat']  = 1

sepsis_ICU.columns 
sepsis_ICU.columns = ['subject_id', 'hadm_id', 'stay_id', 'gender', 'dod', 'admittime',
       'dischtime', 'los_hospital', 'admission_age', 'hospital_expire_flag',
       'icu_intime', 'icu_outtime', 'los_icu', 'suspected_infection_time',
       'sofa_score', 'sepsis', 'lactate', 'ph', 'po2', 'pco2', 'hematocrit',
       'hemoglobin', 'hemoglobin_max', 'platelets', 'wbc', 'bun', 'creatinine',
       'glucose', 'sodium', 'potassium', 'pt', 'ptt', 'bilirubin', 'mean_hb',
       'HR', 'SBP', 'DBP', 'MBP', 'RR', 'T_min', 'T_max', 'spo2', 'weight',
       'height', 'apsiii', 'aki']

### 血管活性药物
vaso = pd.read_csv('D:\hongxiboandsepsis\pressin.csv')
vaso['vaso'] = 1
vaso = vaso.drop(labels = ['vaso_amount'],axis=1)
vaso.columns = ['stay_id', 'vasotime', 'vaso']
vaso['vasotime'] = pd.to_datetime(vaso['vasotime'])
vaso['vaso_rank']= vaso.groupby('stay_id')['vasotime'].rank(method='first',
na_option = 'bottom')
vaso = vaso.sort_values(by=['stay_id','vaso_rank'])
vaso.drop_duplicates(subset=['stay_id'],inplace=True)
### 合并表格
sepsis_ICU =  pd.merge(sepsis_ICU,vaso,on=['stay_id'],how='left')
sepsis_ICU.groupby('vaso').count()
sepsis_ICU['vaso'].fillna(0,inplace=True)
sepsis_ICU['sepsis_shock'] = 0
sepsis_ICU.loc[(sepsis_ICU['vaso']==1) | ((sepsis_ICU['MBP'] < 65) & 
               (sepsis_ICU['lactate'] >=2)),'sepsis_shock'] = 1
sepsis_ICU.groupby('sepsis_shock').count()

### 导入机械通气
venti = pd.read_csv('D:\hongxiboandsepsis\entilation.csv')
venti['ventil'] = 0
venti.columns
venti.loc[venti['ventilation_status'] == 'InvasiveVent','ventil'] = 1
venti.groupby('ventil').count()
venti.sort_values(by=['stay_id','ventil'],ascending=False,inplace=True)
venti.drop_duplicates(subset=['stay_id'],inplace=True)
venti.groupby('ventil').count()
### 合并机械通气
sepsis_ICU = pd.merge(sepsis_ICU,venti,on = ['stay_id'],how='left')
sepsis_ICU.columns
sepsis_ICU = sepsis_ICU.drop(labels = ['vasotime','vaso_rank',
                                       'ventilation_status'],axis =1)
sepsis_ICU.columns
sepsis_ICU.groupby('ventil').count()
### 合并红细胞
trufred = pd.read_csv('D:/hongxiboandsepsis/redblood.csv')
trufred['truRed'] = 1
trufred.sort_values('stay_id',inplace = True)
trufred.drop_duplicates(subset = ['stay_id'],inplace = True)
trufred.columns
trufred =trufred.drop(labels = [ 'starttime', 'endtime', 'storetime',
       'itemid', 'amount', 'amountuom', 'rate', 'rateuom', 'orderid',
       'linkorderid', 'ordercategoryname', 'secondaryordercategoryname',
       'ordercomponenttypedescription', 'ordercategorydescription',
       'patientweight', 'totalamount', 'totalamountuom', 'isopenbag',
       'continueinnextdept', 'cancelreason', 'statusdescription',
       'originalamount', 'originalrate'],axis =1)
sepsis_ICU = pd.merge(sepsis_ICU,trufred,on=['subject_id', 'hadm_id',
                                             'stay_id'],how = 'left')
sepsis_ICU .columns
sepsis_ICU['truRed'].fillna(0,inplace=True)
sepsis_ICU.groupby(['truRed']).count()

### 合并基础疾病
diseas = pd.read_csv('D:/hongxiboandsepsis/charlson.csv')
diseas.columns
diseas = diseas[['subject_id', 'hadm_id', 'congestive_heart_failure',
       'chronic_pulmonary_disease', 'mild_liver_disease',
       'diabetes_without_cc', 'diabetes_with_cc', 'renal_disease',
       'severe_liver_disease', 'charlson_comorbidity_index']]
diseas.columns=['subject_id', 'hadm_id', 'CHF','COPD', 'liver_diseaseA', 'diabetesA',
                 'diabetesB', 'Renal','liver_diseaseB', 'charlson_comorbidity_index']
diseas['Liver'] = 0
diseas.loc[(diseas.liver_diseaseA == 1) |(diseas.liver_diseaseB==1),
          'Liver'] = 1
diseas.groupby(diseas.Liver).count()
diseas['Diabetes'] = 0
diseas.loc[(diseas.diabetesA ==1)|(diseas.diabetesB ==1),'Diabetes'] = 1
diseas = diseas.drop(labels=['liver_diseaseA', 'diabetesA', 'diabetesB', 
                                           'liver_diseaseB'],axis =1)

diseas.duplicated(subset=['hadm_id']).sum()

sepsis_ICU = pd.merge(sepsis_ICU,diseas,how='left',on = ['subject_id', 'hadm_id'])
####剔除基础肾脏、肝脏、呼吸系统疾病患者
#sepsis_ICU = sepsis_ICU[(sepsis_ICU.Liver==0)&(sepsis_ICU.Renal==0)]
#sepsis_ICU.columns

#######'hospital_expire_flag'，'gender','aki','vaso', 'sepsis_shock', 'ventil'转换为分类变量
############一、category的创建及其性质
##对DataFrame指定类型创建
## astype('category')
sepsis_ICU['hospital_expire_flag'] =sepsis_ICU['hospital_expire_flag'].astype('category')
sepsis_ICU['gender'] =sepsis_ICU['gender'].astype('category')
sepsis_ICU['aki'] =sepsis_ICU['aki'].astype('category')
sepsis_ICU['vaso'] =sepsis_ICU['vaso'].astype('category')
sepsis_ICU['sepsis_shock'] =sepsis_ICU['sepsis_shock'].astype('category')
sepsis_ICU['ventil'] =sepsis_ICU['ventil'].astype('category')
sepsis_ICU['truRed'] =sepsis_ICU['truRed'].astype('category')
sepsis_ICU['CHF'] =sepsis_ICU['CHF'].astype('category')
sepsis_ICU['COPD'] =sepsis_ICU['COPD'].astype('category')
sepsis_ICU['Renal'] =sepsis_ICU['Renal'].astype('category')
sepsis_ICU['Liver'] =sepsis_ICU['Liver'].astype('category')
sepsis_ICU['Diabetes'] =sepsis_ICU['Diabetes'].astype('category')


#################缺失值填补##############################
################多重插补发进行多重填补####################
#################导入包
import miceforest as mf ## 导入miceforest 包，如果未安装先终端pip install miceforest
import pandas as pd 
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
### 导入相关函数
from sklearn.neighbors import KNeighborsClassifier,KNeighborsRegressor
from sklearn.ensemble import RandomForestRegressor,RandomForestClassifier
from sklearn.tree import DecisionTreeRegressor
from sklearn.linear_model import LinearRegression
from sklearn.svm import SVR

#####检查相缺失值
sepsis_ICU.isna().sum()
sepsis_ICU = sepsis_ICU.drop(labels=['bilirubin'],axis = 1)

######备份数据
sepsis_ICU1 = sepsis_ICU.copy()
sepsis_ICU1.columns
shijian = sepsis_ICU1.loc[:,['subject_id', 'hadm_id','stay_id','dod','admittime', 'dischtime','los_hospital','icu_intime',
       'icu_outtime', 'los_icu', 'suspected_infection_time']]
sepsis_ICU1 =sepsis_ICU1.drop(labels =['dod','admittime', 'dischtime','los_hospital','icu_intime',
       'icu_outtime', 'los_icu', 'suspected_infection_time'],axis =1 )


###################################多重插补
kernel = mf.ImputationKernel(sepsis_ICU1,datasets=4,
                             save_all_iterations=True,
                             random_state =10,
                             mean_match_candidates=5)  
kernel.mice(iterations=4,n_jobs =-1) #设置迭代次数
completed_data = kernel.complete_data(dataset=3)# 提取第几个数据集
completed_data
###########图形诊断工具
##########插补值得分布 分布分析
kernel.plot_imputed_distributions(wspace = 0.1,hspace =0.1)
kernel

############相关性收敛
kernel.plot_correlations()

##############均值收敛图
kernel.plot_mean_convergence(wspace=0.5,hspace=0.5)

###################异常值处理
sepsis_ICU1 = pd.merge(completed_data,shijian,
 on=['subject_id', 'hadm_id','stay_id'],how='left')

sepsis_ICU1.isna().sum()
sepsis_ICU1.columns
########sofa 异常值处理 > 15
plt.scatter(x = sepsis_ICU1['sofa_score'],y = sepsis_ICU1['hematocrit'])

sepsis_ICU1['sofa_score'][(sepsis_ICU1.sofa_score > 15)].count()
sepsis_ICU1['sofa_score'][(sepsis_ICU1.sofa_score > 15)]=15
######## hematocrit 异常值处理 >55  < 10
sepsis_ICU1['hematocrit'][(sepsis_ICU1.hematocrit > 55)].count()
sepsis_ICU1['hematocrit'][(sepsis_ICU1.hematocrit > 55)] = 55
sepsis_ICU1['hematocrit'][(sepsis_ICU1.hematocrit < 10)].count()
sepsis_ICU1['hematocrit'][(sepsis_ICU1.hematocrit < 10)] = 55


##### platelets 异常值处理
sepsis_ICU1['platelets'][(sepsis_ICU1.platelets > 650)].count()
sepsis_ICU1['platelets'][(sepsis_ICU1.platelets > 650)] = 650
sepsis_ICU1['platelets'][(sepsis_ICU1.platelets < 10)].count()
sepsis_ICU1['platelets'][(sepsis_ICU1.platelets < 10)] = 10
####### wbc 异常值处理
plt.scatter(x = sepsis_ICU1.wbc, y = sepsis_ICU1.bun)
sepsis_ICU1['wbc'][(sepsis_ICU1.wbc > 50)].count()
sepsis_ICU1['wbc'][(sepsis_ICU1.wbc > 50)] = 50
sepsis_ICU1['wbc'][(sepsis_ICU1.wbc < 0)].count()

######  bun 异常值处理
sepsis_ICU1['bun'][(sepsis_ICU1.bun > 150)].count()
sepsis_ICU1['bun'][(sepsis_ICU1.bun > 150)] = 150

##### creatinine 异常值处理
plt.scatter(x=sepsis_ICU1.creatinine, y=sepsis_ICU1.glucose)
sepsis_ICU1['creatinine'][(sepsis_ICU1.creatinine > 12)].count()
sepsis_ICU1['creatinine'][(sepsis_ICU1.creatinine > 12)] = 12
##### glucose 异常值处理
sepsis_ICU1['glucose'][(sepsis_ICU1.glucose > 350)].count()
sepsis_ICU1['glucose'][(sepsis_ICU1.glucose > 350)] = 350
sepsis_ICU1['glucose'][(sepsis_ICU1.glucose < 30)].count()
sepsis_ICU1['glucose'][(sepsis_ICU1.glucose < 30)] = 30

#####  sodium 异常值处理
plt.scatter(x=sepsis_ICU1.sodium, y=sepsis_ICU1.potassium)
sepsis_ICU1['sodium'][(sepsis_ICU1.sodium > 170)].count()
sepsis_ICU1['sodium'][(sepsis_ICU1.sodium > 170)]=170
sepsis_ICU1['sodium'][(sepsis_ICU1.sodium < 120)].count()
sepsis_ICU1['sodium'][(sepsis_ICU1.sodium < 120)]=120

#### potassium 异常值处理
sepsis_ICU1['potassium'][(sepsis_ICU1.potassium > 9.5)].count()
sepsis_ICU1['potassium'][(sepsis_ICU1.potassium > 9.5)] = 9.5


####  pt 异常值处理
sepsis_ICU1.columns
plt.scatter(x=sepsis_ICU1.pt, y=sepsis_ICU1.ptt)
sepsis_ICU1['pt'][(sepsis_ICU1.pt > 90)].count()
sepsis_ICU1['pt'][(sepsis_ICU1.pt > 90)] = 90

#### APTT 异常值处理

####lactate 异常值
plt.scatter(x=sepsis_ICU1.lactate, y=sepsis_ICU1.ph)
sepsis_ICU1['lactate'][(sepsis_ICU1.lactate > 20)].count()
sepsis_ICU1['lactate'][(sepsis_ICU1.lactate > 20)]=20

### ph 异常值处理
sepsis_ICU1['ph'][(sepsis_ICU1.ph < 6.8)].count()
sepsis_ICU1['ph'][(sepsis_ICU1.ph < 6.8)] = 6.8

### PO2 异常值处理
plt.scatter(x=sepsis_ICU1.po2, y=sepsis_ICU1.pco2)
sepsis_ICU1['po2'][(sepsis_ICU1.po2 > 400)].count()
sepsis_ICU1['po2'][(sepsis_ICU1.po2 > 400)] = 400
sepsis_ICU1['po2'][(sepsis_ICU1.po2 < 30)] .count()
sepsis_ICU1['po2'][(sepsis_ICU1.po2 < 30)] = 30

### PCO2 异常值处理
sepsis_ICU1['pco2'][(sepsis_ICU1.pco2 > 130)].count()
sepsis_ICU1['pco2'][(sepsis_ICU1.pco2 > 130)] = 130
sepsis_ICU1['pco2'][(sepsis_ICU1.pco2 < 20)].count()
sepsis_ICU1['pco2'][(sepsis_ICU1.pco2 < 20)]=20

### HR 异常值处理
plt.scatter(x=sepsis_ICU1.HR, y=sepsis_ICU1.MBP)
sepsis_ICU1['HR'][(sepsis_ICU1.HR > 180)].count()
sepsis_ICU1['HR'][(sepsis_ICU1.HR > 180)]=180
sepsis_ICU1['HR'][(sepsis_ICU1.HR < 40)].count()
sepsis_ICU1['HR'][(sepsis_ICU1.HR < 40)]=40
### MBP 异常值处理
sepsis_ICU1['MBP'][(sepsis_ICU1.MBP > 115)].count()
sepsis_ICU1['MBP'][(sepsis_ICU1.MBP > 100)]=115


####RR 异常值处理
plt.scatter(x=sepsis_ICU1.RR, y=sepsis_ICU1.T_max)
sepsis_ICU1['RR'][(sepsis_ICU1.RR > 35)].count()
sepsis_ICU1['RR'][(sepsis_ICU1.RR > 35)] =35
sepsis_ICU1['RR'][(sepsis_ICU1.RR < 10)].count()
sepsis_ICU1['RR'][(sepsis_ICU1.RR < 10)]=10
#### BMI 异常值处理
sepsis_ICU1.columns
sepsis_ICU1['BMI'] = sepsis_ICU1['weight']/sepsis_ICU1['height']/sepsis_ICU1['height'] *10000
plt.scatter(x=sepsis_ICU1.BMI, y=sepsis_ICU1.apsiii)
sepsis_ICU1['BMI'][(sepsis_ICU1.BMI > 70)].count()
sepsis_ICU1['BMI'][(sepsis_ICU1.BMI > 70)]=70
sepsis_ICU1['BMI'][(sepsis_ICU1.BMI < 15)].count()
sepsis_ICU1['BMI'][(sepsis_ICU1.BMI < 15)] = 15

#### apsiiii 异常值处理
plt.scatter(x=sepsis_ICU1.apsiii, y=sepsis_ICU1.T_max)
sepsis_ICU1['apsiii'][(sepsis_ICU1.apsiii > 150)].count()
sepsis_ICU1['apsiii'][(sepsis_ICU1.apsiii > 150)]=150
#### lac 异常值处理
plt.scatter(x=sepsis_ICU1.lactate, y=sepsis_ICU1.T_max)
sepsis_ICU1['lactate'][(sepsis_ICU1.lactate > 15)].count()
sepsis_ICU1['lactate'][(sepsis_ICU1.lactate > 15)]=15


########   红细胞分组
# 根据平均血红蛋白分组
sepsis_ICU1.columns
sepsis_ICU1['Hb'] = sepsis_ICU1.mean_hb

###########绘制28d死亡k-m 曲线
####生成存活时间
#####导入生存时间
###死亡时间减去生存时间，对于存在缺失值，使用出院时间减去住ICU时间填补。
sepsis_ICU1.columns 
sepsis_ICU1.dischtime = pd.to_datetime(sepsis_ICU1.dischtime)
sepsis_ICU1['dod'] = pd.to_datetime(sepsis_ICU1['dod'])
sepsis_ICU1['surtime'] =  (sepsis_ICU1['dod'] - sepsis_ICU1['icu_intime']).astype('timedelta64[h]')
sepsis_ICU1['time'] =  (sepsis_ICU1['dischtime'] - sepsis_ICU1['icu_intime']).astype('timedelta64[h]')
sepsis_ICU1['surtime'] = sepsis_ICU1['surtime'].fillna(sepsis_ICU1.time)
sepsis_ICU1['surtime'] = sepsis_ICU1['surtime'].astype(float)
sepsis_ICU1['surtime'] = sepsis_ICU1.surtime / 24
### 合并院内死亡及院外死亡总数
sepsis_ICU1['death'] = 0
sepsis_ICU1['death'] [sepsis_ICU1.dod.isna() == False] = 1
sepsis_ICU1['death'] [sepsis_ICU1.hospital_expire_flag ==1] = 1
sepsis_ICU1.groupby(sepsis_ICU1['death']).count()
########################
sepsis_ICU1['death28'] = 0
##住院时间<=28天的死亡患者认为 存活时间<28天
sepsis_ICU1['death28'][(sepsis_ICU1.surtime <=28) &(sepsis_ICU1.death ==1)] = 1
sepsis_ICU1.groupby('death28').count()
sepsis_ICU1.groupby('hospital_expire_flag').count()
sepsis_ICU1['surtime'][(sepsis_ICU1.surtime >28)] =28 # 存活时间大于28天设置为28天


## 合并早期输血 24h内
### 合并红细胞

trufred24 = pd.read_csv('D:/hongxiboandsepsis/redblood.csv')
trufred24['truRed24'] = 1
trufred24 .columns
trufred24 = trufred24.drop(labels =['endtime', 'storetime',
       'itemid', 'amount', 'amountuom', 'rate', 'rateuom', 'orderid',
       'linkorderid', 'ordercategoryname', 'secondaryordercategoryname',
       'ordercomponenttypedescription', 'ordercategorydescription',
       'patientweight', 'totalamount', 'totalamountuom', 'isopenbag',
       'continueinnextdept', 'cancelreason', 'statusdescription',
       'originalamount', 'originalrate'],axis = 1)
sepsis_ICU3 = sepsis_ICU1.copy()
sepsis_ICU3 = pd.merge(sepsis_ICU3 ,trufred24,on = ['subject_id','hadm_id','stay_id'],how = 'inner')
sepsis_ICU3.columns
sepsis_ICU3['starttime'] =  pd.to_datetime(sepsis_ICU3['starttime'])
sepsis_ICU3['timedelta2'] = (sepsis_ICU3.starttime -sepsis_ICU3.icu_intime).astype('timedelta64[h]')
sepsis_ICU3['timedelta2'] = sepsis_ICU3.timedelta2.astype(float)
## 仅纳入24小时内输血#
sepsis_ICU3 = sepsis_ICU3[(sepsis_ICU3.timedelta2 <= 24)&
                        (sepsis_ICU3.timedelta2 >= 0)] 
sepsis_ICU3.duplicated(subset=['stay_id']).sum() 
sepsis_ICU3=sepsis_ICU3[['subject_id', 'hadm_id', 'stay_id','truRed24']]
sepsis_ICU3.drop_duplicates(subset=['stay_id'],inplace = True)
sepsis_ICU1 = pd.merge(sepsis_ICU1,sepsis_ICU3,on = ['subject_id', 'hadm_id', 
                                                     'stay_id'],how = 'left')
sepsis_ICU1['truRed24'].fillna(0,inplace = True)

sepsis_ICU1.groupby(['truRed24']).count()

### 导入24小时内最低值平均值
low_hb = pd.read_csv('D:/hongxiboandsepsis/firsthemoglobin.csv')
low_hb.columns
low_hb = low_hb.drop(labels = ['labevent_id','specimen_id', 'itemid','valuenum', 
'valueuom',  'ref_range_lower', 'ref_range_upper', 'flag', 'priority', 
'comments' ],axis = 1)
sepsis_ICUlowhb = sepsis_ICU1.copy()
sepsis_ICUlowhb = pd.merge(sepsis_ICU1, low_hb,on=['subject_id'],how='left')
sepsis_ICUlowhb.columns
sepsis_ICUlowhb['charttime'] =  pd.to_datetime(sepsis_ICUlowhb['charttime'])
## 计算血色素测量时间和入ICU时间差
sepsis_ICUlowhb['timedelat1'] = (sepsis_ICUlowhb.charttime -sepsis_ICUlowhb.icu_intime).astype('timedelta64[h]')
sepsis_ICUlowhb['timedelat1'] = sepsis_ICUlowhb.timedelat1.astype(float)
sepsis_ICUlowhb.head(100)
### 仅包括入ICU24小时内血色素
sepsis_ICUlowhb = sepsis_ICUlowhb[(sepsis_ICUlowhb.timedelat1 <= 24)&
                        (sepsis_ICUlowhb.timedelat1 >=-6)]
### 合并最低值
hb_low = sepsis_ICUlowhb.groupby('subject_id',as_index = False)['value'].min()
hb_low.columns =['subject_id', 'low_hb']
hb_low.dropna(subset=['low_hb'],inplace=True)
sepsis_ICU1 = pd.merge(sepsis_ICU1, hb_low,on=['subject_id'],how = 'left')
sepsis_ICU1.columns

##### 导入HB最大值
### 导入24小时内最大值平均值
high_hb = pd.read_csv('D:/hongxiboandsepsis/firsthemoglobin.csv')
high_hb.columns
high_hb = high_hb.drop(labels = ['labevent_id','specimen_id', 'itemid','valuenum', 
'valueuom',  'ref_range_lower', 'ref_range_upper', 'flag', 'priority', 
'comments' ],axis = 1)
sepsis_ICUhighb = sepsis_ICU1.copy()
sepsis_ICUhighb = pd.merge(sepsis_ICU1, high_hb,on=['subject_id'],how='left')
sepsis_ICUhighb.columns
sepsis_ICUhighb['charttime'] =  pd.to_datetime(sepsis_ICUhighb['charttime'])
## 计算血色素测量时间和入ICU时间差
sepsis_ICUhighb['timedelat1'] = (sepsis_ICUhighb.charttime -sepsis_ICUhighb.icu_intime).astype('timedelta64[h]')
sepsis_ICUhighb['timedelat1'] = sepsis_ICUhighb.timedelat1.astype(float)
sepsis_ICUhighb.head(100)
### 仅包括入ICU24小时内血色素
sepsis_ICUhighb = sepsis_ICUhighb[(sepsis_ICUhighb.timedelat1 <= 24)&
                        (sepsis_ICUhighb.timedelat1 >=-6)]
### 合并最大值
hb_high = sepsis_ICUhighb.groupby('subject_id',as_index = False)['value'].max()
hb_high.columns =['subject_id', 'high_hb']
hb_high.dropna(subset=['high_hb'],inplace=True)
sepsis_ICU1 = pd.merge(sepsis_ICU1, hb_high,on=['subject_id'],how = 'left')
sepsis_ICU1.columns
plt.scatter(x=sepsis_ICU1.hemoglobin, y=sepsis_ICU1.high_hb)
sepsis_ICU1['mean_hb'][(sepsis_ICU1.high_hb > 17)].count()
sepsis_ICU1 = sepsis_ICU1[(sepsis_ICU1.high_hb <= 17)]
sepsis_ICU1['mean_hb'][(sepsis_ICU1.high_hb < 6)].count()
sepsis_ICU1 = sepsis_ICU1[(sepsis_ICU1.high_hb >= 6)]
