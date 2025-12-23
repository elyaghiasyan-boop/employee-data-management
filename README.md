# employee-data-management
The Employee Data Management Project uses Python to collect, clean, analyze, and manage employee data, providing salary stats, departmental insights, random employee selection, and OOP-based management for scalable HR and payroll analytics.



import csv
import random
import numpy as np
import re
from collections import Counter


class Employee:
    def __init__(self, emp_id, name, department, salary, joining_date):
        self.emp_id = emp_id
        self.name = name
        self.department = department
        self._salary = float(salary)
        self.joining_date = joining_date
        self._bonus = 0.0

   
    @property
    def salary(self):
        return self._salary

    @salary.setter
    def salary(self, value):
        if value < 0:
            raise ValueError("Salary cannot be negative.")
        self._salary = value

    
    @property
    def bonus(self):
        return self._bonus

    @bonus.setter
    def bonus(self, value):
        if value < 0:
            raise ValueError("Bonus cannot be negative.")
        self._bonus = value

   
    def display_info(self):
        print(self)

    def apply_raise(self, percent):
        self.salary += self.salary * (percent / 100)
        print(f"{self.name}'s new salary after {percent:.2f}% raise: ${self.salary:.2f}")

    def apply_bonus(self, amount):
        self.bonus += amount
        print(f"{self.name} received a bonus of ${amount:.2f}. Total bonus: ${self.bonus:.2f}")

  
    def __str__(self):
        return f"ID: {self.emp_id}, Name: {self.name}, Dept: {self.department}, " \
               f"Salary: ${self.salary:.2f}, Joined: {self.joining_date}, Bonus: ${self.bonus:.2f}"


class Manager(Employee):
    def __init__(self, emp_id, name, department, salary, joining_date, team_size=0):
        super().__init__(emp_id, name, department, salary, joining_date)
        self.team_size = team_size

    def display_info(self):
        print(f"{super().__str__()}, Team Size: {self.team_size}")

    def apply_bonus(self, amount):
        super().apply_bonus(amount * 1.2)  # 20% extra bonus for managers


def load_data(file_path):
    employees = []
    with open(file_path, newline='', encoding='utf-8') as f:
        reader = csv.DictReader(f)
        for row in reader:
            name_clean = re.sub(r'[^a-zA-Z\s]', '', row['Name'])
            employees.append({
                'ID': row['ID'],
                'Name': name_clean,
                'Department': row['Department'],
                'Salary': float(row['Salary']),
                'JoiningDate': row['JoiningDate']
            })
    return employees

def save_data_csv(employees, file_path):
    keys = ['ID', 'Name', 'Department', 'Salary', 'JoiningDate']
    with open(file_path, 'w', newline='', encoding='utf-8') as f:
        writer = csv.DictWriter(f, fieldnames=keys)
        writer.writeheader()
        for emp in employees:
            writer.writerow({
                'ID': emp.emp_id,
                'Name': emp.name,
                'Department': emp.department,
                'Salary': emp.salary,
                'JoiningDate': emp.joining_date
            })
    print(f"Data saved to {file_path}")

def save_data_txt(employees, file_path):
    with open(file_path, 'w', encoding='utf-8') as f:
        f.write("ID\tName\tDepartment\tSalary\tJoiningDate\n")
        for emp in employees:
            f.write(f"{emp.emp_id}\t{emp.name}\t{emp.department}\t{emp.salary:.2f}\t{emp.joining_date}\n")
    print(f"Data saved to {file_path}")

def create_employees(data, manager_ids=[]):
    emps = []
    for d in data:
        if d['ID'] in manager_ids:
            emps.append(Manager(d['ID'], d['Name'], d['Department'], d['Salary'], d['JoiningDate'], team_size=random.randint(5, 20)))
        else:
            emps.append(Employee(d['ID'], d['Name'], d['Department'], d['Salary'], d['JoiningDate']))
    return emps

def random_employee(employees):
    return random.choice(employees)


def salary_stats(employees):
    salaries = [e.salary for e in employees]
    mean = np.mean(salaries)
    mode = Counter(salaries).most_common(1)[0][0]
    variance = np.var(salaries, ddof=1)
    std_dev = np.std(salaries, ddof=1)
    return {
        'Mean Salary': mean,
        'Mode Salary': mode,
        'Variance': variance,
        'Standard Deviation': std_dev,
        'Highest Salary': max(salaries),
        'Lowest Salary': min(salaries)
    }

def department_summary(employees):
    summary = {}
    for e in employees:
        dept = e.department
        summary.setdefault(dept, []).append(e.salary)
    result = []
    for dept, salaries in summary.items():
        result.append({
            'Department': dept,
            'Count': len(salaries),
            'Mean': np.mean(salaries),
            'Max': max(salaries),
            'Min': min(salaries)
        })
    return result

def random_statistics(employees):
    choices = [
        lambda: f"Random Employee Info: {random_employee(employees)}",
        lambda: f"Random Bonus Assignment: {random_employee(employees).apply_bonus(random.randint(500,2000)) or 'Bonus Applied'}",
        lambda: f"Random Uniform: {random.random():.4f}",
        lambda: f"Random Von Mises: {np.random.vonmises(mu=0,kappa=4):.4f}",
        lambda: f"Random Gamma: ${np.random.gamma(shape=2,scale=500):.2f}",
        lambda: f"Random Beta: {np.random.beta(a=2,b=5):.4f}",
        lambda: f"Random Pareto: {np.random.pareto(a=3):.4f}",
        lambda: f"Random Student's t: {np.random.standard_t(df=5):.4f}"
    ]
    return np.random.choice(choices)()


def main():
    file_path = 'employees.csv'
    data = load_data(file_path)
    employees = create_employees(data, manager_ids=['2','4'])

    print("=== Salary Statistics ===")
    for k, v in salary_stats(employees).items():
        print(f"{k}: {v:.2f}")

    print("\n=== Department Summary ===")
    for d in department_summary(employees):
        print(d)

    print("\n=== Random Employee of the Month ===")
    emp = random_employee(employees)
    emp.display_info()

    print("\n=== Applying 5% Raise to Random Employee ===")
    emp.apply_raise(5)

    print("\n=== Random Statistics ===")
    for _ in range(5):
        print(random_statistics(employees))

    save_data_csv(employees, 'employees_saved.csv')
    save_data_txt(employees, 'employees_saved.txt')

if __name__ == "__main__":
    main()
    
    

        
