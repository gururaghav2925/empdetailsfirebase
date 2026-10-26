
# Ex.No:1 To create a employee details fields and to display the employee details using Firebase Database in Android Studio.


## AIM:

To create and display the employee details using Firebase Database in Android Studio.

## EQUIPMENTS REQUIRED:

Android Studio(Min.required Artic Fox)

## ALGORITHM:

Step 1: Open Android Stdio and then click on File -> New -> New project.

Step 2: Then type the Application name as HelloWorld and click Next. 

Step 3: Then select the Minimum SDK as shown below and click Next.

Step 4: Then select the Empty Activity and click Next. Finally click Finish.

Step 5: Design layout in activity_main.xml.

Step 6: Display the employee details in MainActivity file.

Step 7: Save and run the application.

## PROGRAM:
```text
Program to print the DatabaseTable using the firebasedatabase”.
Developed by: GuruRagav Ponjeevith V
Registeration Number: 212223220027
```

### MainActivity.java
```java
package com.example.samplefirebase;

import android.os.Bundle;
import android.text.TextUtils;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import androidx.activity.EdgeToEdge;
import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import com.google.firebase.database.DataSnapshot;
import com.google.firebase.database.DatabaseError;
import com.google.firebase.database.DatabaseReference;
import com.google.firebase.database.FirebaseDatabase;
import com.google.firebase.database.ValueEventListener;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

    private EditText editName, editAge, editSalary;
    private Button buttonSave;
    private RecyclerView recyclerView;
    
    private EmployeeAdapter adapter;
    private DatabaseReference databaseReference;
    private List<Employee> employeeList;
    
    private String currentEditingId = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        EdgeToEdge.enable(this);
        setContentView(R.layout.activity_main);
        
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main), (v, insets) -> {
            Insets systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars());
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom);
            return insets;
        });

        // 1. Initialize Firebase Reference to "employees" node
        databaseReference = FirebaseDatabase.getInstance().getReference("employees");

        // 2. Initialize UI components
        editName = findViewById(R.id.editName);
        editAge = findViewById(R.id.editAge);
        editSalary = findViewById(R.id.editSalary);
        buttonSave = findViewById(R.id.buttonSave);
        recyclerView = findViewById(R.id.recyclerView);

        // 3. Setup RecyclerView
        recyclerView.setLayoutManager(new LinearLayoutManager(this));
        employeeList = new ArrayList<>();
        
        adapter = new EmployeeAdapter(new EmployeeAdapter.OnEmployeeClickListener() {
            @Override
            public void onEditClick(Employee employee) {
                // Populate fields with employee data to edit
                editName.setText(employee.name);
                editAge.setText(String.valueOf(employee.age));
                editSalary.setText(String.valueOf(employee.salary));
                
                // Track which employee we are updating
                currentEditingId = employee.id;
                buttonSave.setText("Update Employee");
            }

            @Override
            public void onDeleteClick(Employee employee) {
                // Remove task from Firebase
                if (employee.id != null) {
                    databaseReference.child(employee.id).removeValue();
                    Toast.makeText(MainActivity.this, "Employee deleted", Toast.LENGTH_SHORT).show();
                }
            }
        });
        recyclerView.setAdapter(adapter);

        // 4. Handle "Save/Update" Button Click
        buttonSave.setOnClickListener(v -> saveEmployee());

        // 5. Read data from Firebase Database in real-time
        databaseReference.addValueEventListener(new ValueEventListener() {
            @Override
            public void onDataChange(@NonNull DataSnapshot snapshot) {
                employeeList.clear(); 
                for (DataSnapshot dataSnapshot : snapshot.getChildren()) {
                    Employee employee = dataSnapshot.getValue(Employee.class);
                    if (employee != null) {
                        employeeList.add(employee);
                    }
                }
                adapter.setEmployeeList(employeeList);
            }

            @Override
            public void onCancelled(@NonNull DatabaseError error) {
                Toast.makeText(MainActivity.this, "Failed to load employees: " + error.getMessage(), Toast.LENGTH_SHORT).show();
            }
        });
    }

    private void saveEmployee() {
        String name = editName.getText().toString().trim();
        String ageStr = editAge.getText().toString().trim();
        String salaryStr = editSalary.getText().toString().trim();

        if (TextUtils.isEmpty(name) || TextUtils.isEmpty(ageStr) || TextUtils.isEmpty(salaryStr)) {
            Toast.makeText(this, "Please fill out all fields", Toast.LENGTH_SHORT).show();
            return;
        }

        int age;
        double salary;
        try {
            age = Integer.parseInt(ageStr);
            salary = Double.parseDouble(salaryStr);
        } catch (NumberFormatException e) {
            Toast.makeText(this, "Invalid number format", Toast.LENGTH_SHORT).show();
            return;
        }

        if (currentEditingId == null) {
            // ADD NEW EMPLOYEE
            String id = databaseReference.push().getKey();
            if (id != null) {
                Employee employee = new Employee(id, name, age, salary);
                databaseReference.child(id).setValue(employee);
                Toast.makeText(this, "Employee added successfully", Toast.LENGTH_SHORT).show();
            }
        } else {
            // UPDATE EXISTING EMPLOYEE
            Employee employee = new Employee(currentEditingId, name, age, salary);
            databaseReference.child(currentEditingId).setValue(employee);
            Toast.makeText(this, "Employee updated successfully", Toast.LENGTH_SHORT).show();
            
            // Reset state back to Add mode
            currentEditingId = null;
            buttonSave.setText("Add Employee");
        }

        // Clear input fields
        editName.setText("");
        editAge.setText("");
        editSalary.setText("");
    }
}

```

### activity_main.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Employee Details"
        android:textSize="22sp"
        android:textStyle="bold"
        android:layout_marginBottom="16dp"/>

    <EditText
        android:id="@+id/editName"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Name"
        android:inputType="textPersonName" />

    <EditText
        android:id="@+id/editAge"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Age"
        android:inputType="number" />

    <EditText
        android:id="@+id/editSalary"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Salary"
        android:inputType="numberDecimal" />

    <Button
        android:id="@+id/buttonSave"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        android:text="Add Employee" />

    <View
        android:layout_width="match_parent"
        android:layout_height="1dp"
        android:layout_marginTop="16dp"
        android:layout_marginBottom="16dp"
        android:background="#CCCCCC" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

</LinearLayout>

```

### Employee.java
```java
package com.example.samplefirebase;

public class Employee {
    public String id;
    public String name;
    public int age;
    public double salary;

    // Default constructor required for Firebase
    public Employee() {
    }

    public Employee(String id, String name, int age, double salary) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.salary = salary;
    }
}

```

### EmployeeAdapter.java
```java
package com.example.samplefirebase;

import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageButton;
import android.widget.TextView;
import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;
import java.util.ArrayList;
import java.util.List;

public class EmployeeAdapter extends RecyclerView.Adapter<EmployeeAdapter.EmployeeViewHolder> {

    public interface OnEmployeeClickListener {
        void onEditClick(Employee employee);
        void onDeleteClick(Employee employee);
    }

    private List<Employee> employeeList = new ArrayList<>();
    private final OnEmployeeClickListener listener;

    public EmployeeAdapter(OnEmployeeClickListener listener) {
        this.listener = listener;
    }

    public void setEmployeeList(List<Employee> employeeList) {
        this.employeeList = employeeList;
        notifyDataSetChanged();
    }

    @NonNull
    @Override
    public EmployeeViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_employee, parent, false);
        return new EmployeeViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull EmployeeViewHolder holder, int position) {
        Employee employee = employeeList.get(position);
        
        holder.textName.setText(employee.name);
        holder.textAge.setText("Age: " + employee.age);
        holder.textSalary.setText("Salary: $" + employee.salary);

        holder.buttonEdit.setOnClickListener(v -> listener.onEditClick(employee));
        holder.buttonDelete.setOnClickListener(v -> listener.onDeleteClick(employee));
    }

    @Override
    public int getItemCount() {
        return employeeList.size();
    }

    static class EmployeeViewHolder extends RecyclerView.ViewHolder {
        TextView textName, textAge, textSalary;
        ImageButton buttonEdit, buttonDelete;

        public EmployeeViewHolder(@NonNull View itemView) {
            super(itemView);
            textName = itemView.findViewById(R.id.textName);
            textAge = itemView.findViewById(R.id.textAge);
            textSalary = itemView.findViewById(R.id.textSalary);
            buttonEdit = itemView.findViewById(R.id.buttonEdit);
            buttonDelete = itemView.findViewById(R.id.buttonDelete);
        }
    }
}

```

### colors.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="black">#FF000000</color>
    <color name="white">#FFFFFFFF</color>
</resources>
```


## OUTPUT

<img width="1535" height="863" alt="image" src="https://github.com/user-attachments/assets/27d49b2e-f50c-4594-8cec-2637fb5c7a0c" />


<img width="1535" height="823" alt="image" src="https://github.com/user-attachments/assets/d616e069-0978-4cad-ada7-209aa4528752" />









## RESULT
Thus a Simple Android Application create a firebase database and to display the employee details using Firbase Real Time Database in Android Studio is developed and executed successfully.
