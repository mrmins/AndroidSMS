# How to read the list of contacts and send SMS

For `Android` development we have to consider two things.
1. The Android version installed in each device
2. Request the permissions in the `Manifest` file.

For this case, the first step is, request the permissions in the `Manifest.xml` file, using the next lines:

```
<uses-permission android:name="android.permission.SEND_SMS"/>
<uses-permission android:name="android.permission.READ_CONTACTS" />
```

The first one, will allow you to **send SMS**, but you have to define the phone number to send it.

The second one, will allow you **read the contacts** on the phone (not linked in cloud).


In your `Activity`, `Fragment` or `Class` you might define an access code for those permissions

```
private int PERMISSIONS_REQUEST_RECEIVE_SMS = 130;
private static final int PERMISSIONS_REQUEST_READ_CONTACTS = 100;
```
    
The number `100` and `130` aren't mandatory, but will be useful at the moment to identify the requested permission.


**First one, we going to read the contacts.**
You have to be clear about, in some API version, you don't have to request permission to use some devices features, like to read contacts, that is the reason you have to identify the API version using the next code:

```
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M && context.checkSelfPermission(Manifest.permission.READ_CONTACTS) != PackageManager.PERMISSION_GRANTED) {
  requestPermissions(new String[]{Manifest.permission.READ_CONTACTS}, PERMISSIONS_REQUEST_READ_CONTACTS);
} else {
  //You have the rights.
  getContactNames();
}
```

The version codes higher than `Build.VERSION_CODES.M` the user needs to grant the permissions. And to read the contacts, you can use the method `getContactNames()`:

```
private List<PhoneContactModel> getContactNames() {
        List<PhoneContactModel> contacts = new ArrayList<>();
        // Get the ContentResolver
        ContentResolver cr = context.getContentResolver();
        // Get the Cursor of all the contacts
        Cursor cursor = cr.query(ContactsContract.Contacts.CONTENT_URI, null, null, null, null);
        // Move the cursor to first. Also check whether the cursor is empty or not.
        if (cursor.moveToFirst()) {
            // Iterate through the cursor
            do {
                // Get the contacts name
                String name = cursor.getString(cursor.getColumnIndex(ContactsContract.Contacts.DISPLAY_NAME));
                String id = cursor.getString(cursor.getColumnIndex(ContactsContract.Contacts._ID));
                if (cursor.getInt(cursor.getColumnIndex(ContactsContract.Contacts.HAS_PHONE_NUMBER)) > 0) {
                    Cursor pCur = cr.query(
                            ContactsContract.CommonDataKinds.Phone.CONTENT_URI,
                            null,
                            ContactsContract.CommonDataKinds.Phone.CONTACT_ID + " = ?",
                            new String[]{ id}, null);
                    while (pCur.moveToNext()) {
                        String phoneNo = pCur.getString(pCur.getColumnIndex( ContactsContract.CommonDataKinds.Phone.NUMBER));
                        contacts.add(new PhoneContactModel(id, name, phoneNo) );
                    }
                    pCur.close();
                }
            } while (cursor.moveToNext());
        }
        // Close the curosor
        cursor.close();
        return contacts;
    }
```

Once, you have the list of contacts, you can build a UI to get the single or multiple contacts that you need to send a SMS.
Now, to request the permission to send SMS, you need a code very similar to the previous one, like this:

```
if (requestCode == PERMISSIONS_REQUEST_RECEIVE_SMS) {
  if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
} else {
  Toast.makeText(context, "Until you grant the permission, we canot send the SMS", Toast.LENGTH_SHORT).show();
}
```
And the code to send a SMS is simple like that:

```
SmsManager smsManager = SmsManager.getDefault();
smsManager.sendTextMessage("+10000001231", null, "Visit me. https://github.com/mrmins" null, null);
```

Hope it helps, folks!




