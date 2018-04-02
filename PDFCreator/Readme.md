***This is very simple tutorial which will tell you how to create a simple PDF Fie creator app. So let's start with the tutorial:***

**Firstly put this code in your build.gradle(Module:app) File:**

		//PDF Creator
		implementation 'com.itextpdf:itextg:5.5.10'
		


**Now we will create MainActivity.java file:**

		package pdf.developer.aero;

		import android.Manifest;
		import android.content.DialogInterface;
		import android.content.Intent;
		import android.content.pm.PackageManager;
		import android.net.Uri;
		import android.os.Build;
		import android.os.Bundle;
		import android.os.Environment;
		import android.support.annotation.NonNull;
		import android.support.v4.app.ActivityCompat;
		import android.support.v7.app.AlertDialog;
		import android.support.v7.app.AppCompatActivity;
		import android.util.Log;
		import android.view.View;
		import android.widget.Button;
		import android.widget.EditText;
		import android.widget.Toast;
		import com.itextpdf.text.Document;
		import com.itextpdf.text.DocumentException;
		import com.itextpdf.text.Paragraph;
		import com.itextpdf.text.pdf.PdfWriter;
		import java.io.File;
		import java.io.FileNotFoundException;
		import java.io.FileOutputStream;
		import java.io.OutputStream;
		import java.util.List;

		public class MainActivity extends AppCompatActivity {
			private static final String TAG = "PdfCreatorActivity";
			private EditText  mContentEditText;
			private File pdfFile;
			final private int REQUEST_CODE_ASK_PERMISSIONS = 111;

			@Override
			protected void onCreate(Bundle savedInstanceState) {
				super.onCreate(savedInstanceState);
				setContentView(R.layout.activity_main);

				mContentEditText = findViewById(R.id.edit_text_content);
				Button mCreateButton = findViewById(R.id.button_create);
				mCreateButton.setOnClickListener(new View.OnClickListener() {
					@Override
					public void onClick(View v) {
						if (mContentEditText.getText().toString().isEmpty()){
							mContentEditText.setError("Body is empty");
							mContentEditText.requestFocus();
							return;
						}
						try {
							createPdfWrapper();
						} catch (FileNotFoundException e) {
							e.printStackTrace();
						} catch (DocumentException e) {
							e.printStackTrace();
						}
					}
				});

			}
			private void createPdfWrapper() throws FileNotFoundException,DocumentException{

				int hasWriteStoragePermission = ActivityCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE);
				if (hasWriteStoragePermission != PackageManager.PERMISSION_GRANTED) {

					if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
						if (!shouldShowRequestPermissionRationale(Manifest.permission.WRITE_CONTACTS)) {
							showMessageOKCancel(
									new DialogInterface.OnClickListener() {
										@Override
										public void onClick(DialogInterface dialog, int which) {
											if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
												requestPermissions(new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE},
														REQUEST_CODE_ASK_PERMISSIONS);
											}
										}
									});
							return;
						}

						requestPermissions(new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE},
								REQUEST_CODE_ASK_PERMISSIONS);
					}
				}else {
					createPdf();
				}
			}
			@Override
			public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
				switch (requestCode) {
					case REQUEST_CODE_ASK_PERMISSIONS:
						if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
							// Permission Granted
							try {
								createPdfWrapper();
							} catch (FileNotFoundException e) {
								e.printStackTrace();
							} catch (DocumentException e) {
								e.printStackTrace();
							}
						} else {
							// Permission Denied
							Toast.makeText(this, "WRITE_EXTERNAL Permission Denied", Toast.LENGTH_SHORT)
									.show();
						}
						break;
					default:
						super.onRequestPermissionsResult(requestCode, permissions, grantResults);
				}
			}
			private void showMessageOKCancel(DialogInterface.OnClickListener okListener) {
				new AlertDialog.Builder(this)
						.setMessage("You need to allow access to Storage")
						.setPositiveButton("OK", okListener)
						.setNegativeButton("Cancel", null)
						.create()
						.show();
			}

			private void createPdf() throws FileNotFoundException, DocumentException {

				File docsFolder = new File(Environment.getExternalStorageDirectory() + "/Documents");
				if (!docsFolder.exists()) {
					//noinspection ResultOfMethodCallIgnored
					docsFolder.mkdir();
					Log.i(TAG, "Created a new directory for PDF");
				}

				pdfFile = new File(docsFolder.getAbsolutePath(),"HelloWorld.pdf");
				OutputStream output = new FileOutputStream(pdfFile);
				Document document = new Document();
				PdfWriter.getInstance(document, output);
				document.open();
				document.add(new Paragraph(mContentEditText.getText().toString()));

				document.close();
				previewPdf();

			}

			private void previewPdf() {

				PackageManager packageManager = getPackageManager();
				Intent testIntent = new Intent(Intent.ACTION_VIEW);
				testIntent.setType("application/pdf");
				List list = packageManager.queryIntentActivities(testIntent, PackageManager.MATCH_DEFAULT_ONLY);
				if (list.size() > 0) {
					Intent intent = new Intent();
					intent.setAction(Intent.ACTION_VIEW);
					Uri uri = Uri.fromFile(pdfFile);
					intent.setDataAndType(uri, "application/pdf");

					startActivity(intent);
				}else{
					Toast.makeText(this,"Download Adobe Reader to see the generated PDF",Toast.LENGTH_SHORT).show();
				}
			}
		}

**Now create activity_main layout file res -> layout folder:**

		<?xml version="1.0" encoding="utf-8"?>
		<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
			android:orientation="vertical"
			android:layout_width="match_parent"
			android:layout_height="match_parent"
			android:gravity="top">
			<EditText
				android:id="@+id/edit_text_content"
				android:layout_width="match_parent"
				android:layout_height="wrap_content"
				android:gravity="top"
				android:hint="@string/write"
				android:inputType="textMultiLine"
				android:layout_margin="5dp"
				android:padding="20dp"
				android:lines="5">
			</EditText>
			<Button
				android:id="@+id/button_create"
				android:layout_width="wrap_content"
				android:layout_gravity="center_horizontal"
				android:layout_height="wrap_content"
				android:text="@string/createpdf"/>
		</LinearLayout>	

**This will be strings file:**

		<resources>
			<string name="app_name">PDFCreator</string>
			<string name="write">Write the content</string>
			<string name="createpdf">Create PDF</string>
		</resources>
		
**And Lastly This will be your Manifest File:**

    <?xml version="1.0" encoding="utf-8"?>
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="pdf.developer.aero">

        <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>

        <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>

        <application
            android:allowBackup="true"
            android:icon="@mipmap/ic_launcher"
            android:label="@string/app_name"
            android:roundIcon="@mipmap/ic_launcher_round"
            android:supportsRtl="true"
            android:theme="@style/AppTheme">
            <activity android:name=".MainActivity">
                <intent-filter>
                    <action android:name="android.intent.action.MAIN" />

                    <category android:name="android.intent.category.LAUNCHER" />
                </intent-filter>
            </activity>
        </application>

    </manifest>

**Output:**

![alt text]()