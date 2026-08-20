# PMD-Workshop-1

## AIM

To develop an Android application using SQLite Database to store country names and their respective currencies in a ListView and perform CRUD operations.

## ALGORITHM

1. Create an Android project.
2. Create the following five Java classes:
   - `DatabaseHelper.java`
   - `DBManager.java`
   - `CountryListActivity.java`
   - `AddCountryActivity.java`
   - `ModifyCountryActivity.java`
3. Create an SQLite database named `CountryDB`.
4. Create a table named `country` with `id`, `country`, and `currency` fields.
5. Display country names and currencies using a ListView.
6. Add new country and currency records to the database.
7. Select a record from the ListView to modify or delete it.
8. Update the selected record in the SQLite database.
9. Delete the selected record from the SQLite database.
10. Refresh the ListView after every database operation.

## PROGRAM

### DatabaseHelper.java

```
package com.example.countrycurrency;

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

public class DatabaseHelper extends SQLiteOpenHelper {

    private static final String DATABASE_NAME = "CountryDB.db";
    private static final int DATABASE_VERSION = 1;

    public static final String TABLE_NAME = "country";
    public static final String COL_ID = "id";
    public static final String COL_COUNTRY = "country";
    public static final String COL_CURRENCY = "currency";

    public DatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {

        String query = "CREATE TABLE " + TABLE_NAME + " (" +
                COL_ID + " INTEGER PRIMARY KEY AUTOINCREMENT, " +
                COL_COUNTRY + " TEXT, " +
                COL_CURRENCY + " TEXT)";

        db.execSQL(query);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

        db.execSQL("DROP TABLE IF EXISTS " + TABLE_NAME);
        onCreate(db);
    }
}

```
## DBManager.java
```
package com.example.countrycurrency;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;

public class DBManager {

    private DatabaseHelper dbHelper;
    private SQLiteDatabase database;

    public DBManager(Context context) {
        dbHelper = new DatabaseHelper(context);
    }

    public void open() {
        database = dbHelper.getWritableDatabase();
    }

    public void close() {
        dbHelper.close();
    }

    // CREATE
    public long addCountry(String country, String currency) {

        ContentValues values = new ContentValues();

        values.put(DatabaseHelper.COL_COUNTRY, country);
        values.put(DatabaseHelper.COL_CURRENCY, currency);

        return database.insert(
                DatabaseHelper.TABLE_NAME,
                null,
                values
        );
    }

    // READ
    public Cursor getAllCountries() {

        return database.query(
                DatabaseHelper.TABLE_NAME,
                null,
                null,
                null,
                null,
                null,
                DatabaseHelper.COL_ID + " DESC"
        );
    }

    // UPDATE
    public int updateCountry(int id, String country, String currency) {

        ContentValues values = new ContentValues();

        values.put(DatabaseHelper.COL_COUNTRY, country);
        values.put(DatabaseHelper.COL_CURRENCY, currency);

        return database.update(
                DatabaseHelper.TABLE_NAME,
                values,
                DatabaseHelper.COL_ID + "=?",
                new String[]{String.valueOf(id)}
        );
    }

    // DELETE
    public int deleteCountry(int id) {

        return database.delete(
                DatabaseHelper.TABLE_NAME,
                DatabaseHelper.COL_ID + "=?",
                new String[]{String.valueOf(id)}
        );
    }
}
```

## CountryListActivity.java
```
package com.example.countrycurrency;

import android.content.Intent;
import android.database.Cursor;
import android.os.Bundle;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.ListView;

import androidx.appcompat.app.AppCompatActivity;

import java.util.ArrayList;

public class CountryListActivity extends AppCompatActivity {

    ListView listView;
    Button addButton;

    DBManager dbManager;

    ArrayList<String> countryList;
    ArrayList<Integer> idList;

    ArrayAdapter<String> adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_country_list);

        listView = findViewById(R.id.listView);
        addButton = findViewById(R.id.addButton);

        dbManager = new DBManager(this);
        dbManager.open();

        loadCountries();

        addButton.setOnClickListener(v -> {

            Intent intent = new Intent(
                    CountryListActivity.this,
                    AddCountryActivity.class
            );

            startActivity(intent);
        });

        listView.setOnItemClickListener(
                (parent, view, position, id) -> {

                    Intent intent = new Intent(
                            CountryListActivity.this,
                            ModifyCountryActivity.class
                    );

                    intent.putExtra(
                            "id",
                            idList.get(position)
                    );

                    startActivity(intent);
                }
        );
    }

    private void loadCountries() {

        countryList = new ArrayList<>();
        idList = new ArrayList<>();

        Cursor cursor = dbManager.getAllCountries();

        while (cursor.moveToNext()) {

            int id = cursor.getInt(
                    cursor.getColumnIndexOrThrow(
                            DatabaseHelper.COL_ID
                    )
            );

            String country = cursor.getString(
                    cursor.getColumnIndexOrThrow(
                            DatabaseHelper.COL_COUNTRY
                    )
            );

            String currency = cursor.getString(
                    cursor.getColumnIndexOrThrow(
                            DatabaseHelper.COL_CURRENCY
                    )
            );

            idList.add(id);

            countryList.add(
                    country + " - " + currency
            );
        }

        cursor.close();

        adapter = new ArrayAdapter<>(
                this,
                android.R.layout.simple_list_item_1,
                countryList
        );

        listView.setAdapter(adapter);
    }

    @Override
    protected void onResume() {
        super.onResume();

        if (dbManager != null) {
            loadCountries();
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        if (dbManager != null) {
            dbManager.close();
        }
    }
}
```
## AddCountryActivity.java
```
package com.example.countrycurrency;

import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

public class AddCountryActivity extends AppCompatActivity {

    EditText countryEditText;
    EditText currencyEditText;
    Button saveButton;

    DBManager dbManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_add_country);

        countryEditText = findViewById(R.id.countryEditText);
        currencyEditText = findViewById(R.id.currencyEditText);
        saveButton = findViewById(R.id.saveButton);

        dbManager = new DBManager(this);
        dbManager.open();

        saveButton.setOnClickListener(v -> {

            String country =
                    countryEditText.getText().toString().trim();

            String currency =
                    currencyEditText.getText().toString().trim();

            if (country.isEmpty() || currency.isEmpty()) {

                Toast.makeText(
                        this,
                        "Enter all details",
                        Toast.LENGTH_SHORT
                ).show();

                return;
            }

            long result =
                    dbManager.addCountry(country, currency);

            if (result != -1) {

                Toast.makeText(
                        this,
                        "Country Added Successfully",
                        Toast.LENGTH_SHORT
                ).show();

                finish();

            } else {

                Toast.makeText(
                        this,
                        "Failed to Add Country",
                        Toast.LENGTH_SHORT
                ).show();
            }
        });
    }
}
```
## ModifyCountryActivity.java
```
package com.example.countrycurrency;

import android.database.Cursor;
import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

public class ModifyCountryActivity extends AppCompatActivity {

    EditText countryEditText;
    EditText currencyEditText;

    Button updateButton;
    Button deleteButton;

    DBManager dbManager;

    int countryId;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_modify_country);

        countryEditText = findViewById(R.id.countryEditText);
        currencyEditText = findViewById(R.id.currencyEditText);

        updateButton = findViewById(R.id.updateButton);
        deleteButton = findViewById(R.id.deleteButton);

        countryId = getIntent().getIntExtra("id", -1);

        dbManager = new DBManager(this);
        dbManager.open();

        loadCountry();

        updateButton.setOnClickListener(v -> {

            String country =
                    countryEditText.getText().toString().trim();

            String currency =
                    currencyEditText.getText().toString().trim();

            int result = dbManager.updateCountry(
                    countryId,
                    country,
                    currency
            );

            if (result > 0) {

                Toast.makeText(
                        this,
                        "Country Updated Successfully",
                        Toast.LENGTH_SHORT
                ).show();

                finish();
            }
        });

        deleteButton.setOnClickListener(v -> {

            int result =
                    dbManager.deleteCountry(countryId);

            if (result > 0) {

                Toast.makeText(
                        this,
                        "Country Deleted Successfully",
                        Toast.LENGTH_SHORT
                ).show();

                finish();
            }
        });
    }

    private void loadCountry() {

        Cursor cursor = dbManager.getAllCountries();

        while (cursor.moveToNext()) {

            int id = cursor.getInt(
                    cursor.getColumnIndexOrThrow(
                            DatabaseHelper.COL_ID
                    )
            );

            if (id == countryId) {

                String country = cursor.getString(
                        cursor.getColumnIndexOrThrow(
                                DatabaseHelper.COL_COUNTRY
                        )
                );

                String currency = cursor.getString(
                        cursor.getColumnIndexOrThrow(
                                DatabaseHelper.COL_CURRENCY
                        )
                );

                countryEditText.setText(country);
                currencyEditText.setText(currency);

                break;
            }
        }

        cursor.close();
    }
}
```
## activity_country_list.xml
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <Button
        android:id="@+id/addButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Add Country"/>

    <ListView
        android:id="@+id/listView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_marginTop="10dp"/>

</LinearLayout>
```
## activity_add_country.xml
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="20dp">

    <EditText
        android:id="@+id/countryEditText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Country Name"/>

    <EditText
        android:id="@+id/currencyEditText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Currency"/>

    <Button
        android:id="@+id/saveButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Save"/>

</LinearLayout>
```
## activity_modify_country.xml
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="20dp">

    <EditText
        android:id="@+id/countryEditText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Country Name"/>

    <EditText
        android:id="@+id/currencyEditText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Currency"/>

    <Button
        android:id="@+id/updateButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Update"/>

    <Button
        android:id="@+id/deleteButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Delete"/>

</LinearLayout>
```

## Output

<img width="1080" height="2160" alt="01_country_list" src="https://github.com/user-attachments/assets/c4566ad9-82fd-4052-ad2e-46390c65e3ee" />

<img width="1080" height="2160" alt="02_add_country" src="https://github.com/user-attachments/assets/f5abc236-193c-4b3e-b20a-f1220647d360" />

<img width="1080" height="2160" alt="03_modify_country" src="https://github.com/user-attachments/assets/aa03f726-78ad-48a1-8770-f5828e328fe4" />

<img width="1080" height="2160" alt="04_after_update" src="https://github.com/user-attachments/assets/613ccc5a-ddc4-40dd-a846-3bb3fd24a5d3" />

<img width="1080" height="2160" alt="05_after_delete" src="https://github.com/user-attachments/assets/7b65c92a-ea7c-48c6-ab30-a29953b1fa35" />


## Result
Thus, the Android application was successfully developed using SQLite Database to store country names and their respective currencies in a ListView. CRUD operations such as Create, Read, Update, and Delete were successfully performed.
