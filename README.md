# -------------------------------
# Healthcare Star Schema Data Generator
# -------------------------------

import pandas as pd
import random
from datetime import datetime, timedelta

# -------------------------------
# Helper Functions
# -------------------------------
def random_date(start, end):
    """Generate a random date between start and end"""
    delta = end - start
    random_days = random.randint(0, delta.days)
    return start + timedelta(days=random_days)

def random_choice(choices):
    return random.choice(choices)

# -------------------------------
# Configuration
# -------------------------------
num_fact_records = 50000  # For demonstration; increase to 500000 for large datasets
batch_size = 5000
start_date = datetime(2020, 1, 1)
end_date = datetime(2025, 12, 31)

# -------------------------------
# Dimension Tables Definitions
# -------------------------------

# Dim_Patient
num_patients = 5000
def generate_dim_patient(num_records):
    patients = []
    for i in range(1, num_records+1):
        admission_date = random_date(start_date, end_date)
        discharge_date = admission_date + timedelta(days=random.randint(1, 30))
        row = {
            "Patient_ID": i,
            "First_Name": f"Patient_{i}",
            "Last_Name": f"Last_{i}",
            "Gender": random_choice(["Male", "Female"]),
            "DOB": random_date(datetime(1950,1,1), datetime(2015,12,31)).strftime("%Y-%m-%d"),
            "Admission_Date": admission_date.strftime("%Y-%m-%d"),
            "Discharge_Date": discharge_date.strftime("%Y-%m-%d")
        }
        patients.append(row)
    return pd.DataFrame(patients)

df_dim_patient = generate_dim_patient(num_patients)
df_dim_patient.to_csv("Dim_Patient.csv", index=False)

# Dim_Doctor
num_doctors = 200
df_dim_doctor = pd.DataFrame([{
    "Doctor_ID": i,
    "First_Name": f"Doctor_{i}",
    "Last_Name": f"Last_{i}",
    "Specialization": random_choice(["Cardiology","Neurology","Orthopedics","General","Surgery"]),
    "Email": f"doctor{i}@hospital.com",
    "Phone": f"+91{random.randint(9000000000,9999999999)}",
    "Hire_Date": random_date(datetime(2000,1,1), datetime(2025,1,1)).strftime("%Y-%m-%d")
} for i in range(1, num_doctors+1)])
df_dim_doctor.to_csv("Dim_Doctor.csv", index=False)

# Dim_Department
departments_list = ["Cardiology","Neurology","Orthopedics","General","Surgery","Pediatrics","Radiology"]
df_dim_department = pd.DataFrame([{
    "Department_ID": i+1,
    "Department_Name": name,
    "Head_Of_Department": f"Doctor_{random.randint(1,num_doctors)}",
    "Floor": random.randint(1,10),
    "Phone": f"+91{random.randint(9000000000,9999999999)}",
    "Email": f"{name.lower()}@hospital.com"
} for i,name in enumerate(departments_list)])
df_dim_department.to_csv("Dim_Department.csv", index=False)

# Dim_Hospital
num_hospitals = 20
df_dim_hospital = pd.DataFrame([{
    "Hospital_ID": i,
    "Hospital_Name": f"Hospital_{i}",
    "City": random_choice(["Mumbai","Pune","Delhi","Bangalore","Chennai"]),
    "State": random_choice(["Maharashtra","Karnataka","Delhi","Tamil Nadu"]),
    "Phone": f"+91{random.randint(9000000000,9999999999)}",
    "Capacity": random.randint(50,500),
    "Type": random_choice(["Private","Government"])
} for i in range(1,num_hospitals+1)])
df_dim_hospital.to_csv("Dim_Hospital.csv", index=False)

# Dim_Procedure
num_procedures = 50
df_dim_procedure = pd.DataFrame([{
    "Procedure_ID": i,
    "Procedure_Name": f"Procedure_{i}",
    "Department_ID": random_choice(df_dim_department["Department_ID"].tolist()),
    "Cost": random.randint(5000,50000),
    "Duration_Minutes": random.randint(30,240),
    "Description": f"Description of procedure {i}",
    "Required_Equipment": f"Equipment_{random.randint(1,100)}"
} for i in range(1,num_procedures+1)])
df_dim_procedure.to_csv("Dim_Procedure.csv", index=False)

# Dim_Medication
num_medications = 100
df_dim_medication = pd.DataFrame([{
    "Medication_ID": i,
    "Medication_Name": f"Medication_{i}",
    "Type": random_choice(["Tablet","Injection","Syrup"]),
    "Dose_mg": random.randint(10,500),
    "Manufacturer": f"Pharma_{random.randint(1,50)}",
    "Cost": random.randint(50,2000),
    "Stock_Quantity": random.randint(10,1000)
} for i in range(1,num_medications+1)])
df_dim_medication.to_csv("Dim_Medication.csv", index=False)

# Dim_Lab
num_labs = 50
df_dim_lab = pd.DataFrame([{
    "Lab_ID": i,
    "Lab_Name": f"Lab_{i}",
    "City": random_choice(["Mumbai","Pune","Delhi"]),
    "Phone": f"+91{random.randint(9000000000,9999999999)}",
    "Department_ID": random_choice(df_dim_department["Department_ID"].tolist()),
    "Head_Lab_Technician": f"Tech_{i}",
    "Capacity_Tests_Per_Day": random.randint(50,500)
} for i in range(1,num_labs+1)])
df_dim_lab.to_csv("Dim_Lab.csv", index=False)

# -------------------------------
# Fact Table Generators
# -------------------------------
def generate_fact_visits(idx):
    return {
        "Visit_ID": idx,
        "Patient_ID": random_choice(df_dim_patient["Patient_ID"].tolist()),
        "Doctor_ID": random_choice(df_dim_doctor["Doctor_ID"].tolist()),
        "Department_ID": random_choice(df_dim_department["Department_ID"].tolist()),
        "Hospital_ID": random_choice(df_dim_hospital["Hospital_ID"].tolist()),
        "Visit_Date": random_date(start_date, end_date).strftime("%Y-%m-%d"),
        "Visit_Type": random_choice(["Emergency","Routine","Follow-up"]),
        "Visit_Cost": random.randint(500,5000),
        "Duration_Minutes": random.randint(15,120),
        "Outcome": random_choice(["Recovered","Referred","Admitted"])
    }

def generate_fact_operations(idx):
    return {
        "Operation_ID": idx,
        "Patient_ID": random_choice(df_dim_patient["Patient_ID"].tolist()),
        "Doctor_ID": random_choice(df_dim_doctor["Doctor_ID"].tolist()),
        "Procedure_ID": random_choice(df_dim_procedure["Procedure_ID"].tolist()),
        "Hospital_ID": random_choice(df_dim_hospital["Hospital_ID"].tolist()),
        "Date": random_date(start_date, end_date).strftime("%Y-%m-%d"),
        "Equipment_Used": f"Equipment_{random.randint(1,100)}",
        "Operation_Duration_Minutes": random.randint(60,200),
        "Operation_Cost": random.randint(10000,50000),
        "Outcome_Status": random_choice(["Successful","Complications","Failed"]),
        "Recovery_Time_Days": random.randint(3,14)
    }

def generate_fact_medications(idx):
    return {
        "Prescription_ID": idx,
        "Patient_ID": random_choice(df_dim_patient["Patient_ID"].tolist()),
        "Doctor_ID": random_choice(df_dim_doctor["Doctor_ID"].tolist()),
        "Medication_ID": random_choice(df_dim_medication["Medication_ID"].tolist()),
        "Date": random_date(start_date, end_date).strftime("%Y-%m-%d"),
        "Dosage": f"{random.randint(1,3)} tablet(s) per day",
        "Duration_Days": random.randint(3,14),
        "Prescription_Cost": random.randint(100,2000)
    }

def generate_fact_labtests(idx):
    return {
        "Test_ID": idx,
        "Patient_ID": random_choice(df_dim_patient["Patient_ID"].tolist()),
        "Doctor_ID": random_choice(df_dim_doctor["Doctor_ID"].tolist()),
        "LabTest_ID": random.randint(1,50),
        "Lab_ID": random_choice(df_dim_lab["Lab_ID"].tolist()),
        "Date": random_date(start_date, end_date).strftime("%Y-%m-%d"),
        "Test_Result_Value": round(random.uniform(0.1,15.0),2),
        "Test_Result_Unit": random_choice(["mg/dL","mmHg","IU/L"]),
        "Test_Cost": random.randint(500,5000)
    }

def generate_fact_hospitalizations(idx):
    admission_date = random_date(start_date, end_date)
    discharge_date = admission_date + timedelta(days=random.randint(1,20))
    return {
        "Admission_ID": idx,
        "Patient_ID": random_choice(df_dim_patient["Patient_ID"].tolist()),
        "Doctor_ID": random_choice(df_dim_doctor["Doctor_ID"].tolist()),
        "Department_ID": random_choice(df_dim_department["Department_ID"].tolist()),
        "Hospital_ID": random_choice(df_dim_hospital["Hospital_ID"].tolist()),
        "Room_ID": random.randint(1,100),
        "Admission_Date": admission_date.strftime("%Y-%m-%d"),
        "Discharge_Date": discharge_date.strftime("%Y-%m-%d"),
        "Length_of_Stay_Days": (discharge_date - admission_date).days,
        "Admission_Cost": random.randint(5000,50000),
        "Outcome": random_choice(["Recovered","Referred","Deceased"])
    }

# -------------------------------
# Batch CSV Writing for Fact Tables
# -------------------------------
def write_fact_csv(filename, row_func):
    chunks = num_fact_records // batch_size
    for i in range(chunks):
        data = [row_func(j + i*batch_size) for j in range(batch_size)]
        df = pd.DataFrame(data)
        if i==0:
            df.to_csv(filename, index=False, mode='w')
        else:
            df.to_csv(filename, index=False, mode='a', header=False)
        print(f"Written batch {i+1}/{chunks} to {filename}")

# -------------------------------
# Generate Fact Tables
# -------------------------------
print("Generating Fact_Visits...")
write_fact_csv("Fact_Visits.csv", generate_fact_visits)

print("Generating Fact_Operations...")
write_fact_csv("Fact_Operations.csv", generate_fact_operations)

print("Generating Fact_Medications...")
write_fact_csv("Fact_Medications.csv", generate_fact_medications)

print("Generating Fact_LabTests...")
write_fact_csv("Fact_LabTests.csv", generate_fact_labtests)

print("Generating Fact_Hospitalizations...")
write_fact_csv("Fact_Hospitalizations.csv", generate_fact_hospitalizations)

print("All fact tables and dimension tables generated successfully!")
