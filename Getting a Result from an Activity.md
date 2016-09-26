原文地址：[http://android.xsoftlab.net/training/basics/intents/result.html](http://android.xsoftlab.net/training/basics/intents/result.html)

启动其它Activity并不是单方向的。你也可以启动其它Activity然后接收返回结果。如果要接收结果，应该调用[startActivityForResult()](http://android.xsoftlab.net/reference/android/app/Activity.html#startActivityForResult(android.content.Intent,%20int))而不是[startActivity()](http://android.xsoftlab.net/reference/android/app/Activity.html#startActivity(android.content.Intent))。

举个例子，APP可以启动拍照APP然后接收捕捉到的图像作为结果。或者，你可能会启动一个People APP让用户选择一个联系人，然后接收联系人的详细信息作为返回结果。

当前，响应的activity必须要被设计为可以返回结果。当这样被设计了，它会发送另一个Intent对象作为结果。你的Activity会在[onActivityResult()](http://android.xsoftlab.net/reference/android/app/Activity.html#onActivityResult(int,%20int,%20android.content.Intent))回调方法中接收到它。

> **Note:**当你在调用[startActivityForResult()](http://android.xsoftlab.net/reference/android/app/Activity.html#startActivityForResult(android.content.Intent,%20int))方法的时候可以使用显式或者隐式的意图。当启动你自己的Activity接收结果时，你应该使用显式意图来确保你可以接收到期望的结果。

##启动Activity
当在启动Activity的时候并没有特别指明Intent对象的结果，但是你还是需要传一个附加整型值给[startActivityForResult()](http://android.xsoftlab.net/reference/android/app/Activity.html#startActivityForResult(android.content.Intent,%20int))方法。

这个整型的参数被称为"request code",它用来指明你的请求。当你接收到Intent的结果时，回调方法会提供相同的请求码，以便于APP可以适当的确认结果并决定怎么处理它。

举个例子，这里展示了如何启动一个activity，并且允许用户选择一个联系人：
```java
static final int PICK_CONTACT_REQUEST = 1;  // The request code
...
private void pickContact() {
    Intent pickContactIntent = new Intent(Intent.ACTION_PICK, Uri.parse("content://contacts"));
    pickContactIntent.setType(Phone.CONTENT_TYPE); // Show user only contacts w/ phone numbers
    startActivityForResult(pickContactIntent, PICK_CONTACT_REQUEST);
}
```

##接收结果
当用户操作完成并返回的时候，系统会调用activity的[onActivityResult()](http://android.xsoftlab.net/reference/android/app/Activity.html#onActivityResult(int,%20int,%20android.content.Intent))方法，这个方法包含了三个参数：

- 使用startActivityForResult()方法时传递的请求码
- 第二个activity指定的结果码，这里可能是[RESULT_OK](http://android.xsoftlab.net/reference/android/app/Activity.html#RESULT_OK)，如果操作完成的话，要不然就是[RESULT_CANCELED](http://android.xsoftlab.net/reference/android/app/Activity.html#RESULT_CANCELED)，如果用户返回了，或者因为别的原因失败了
- 一个携带了结果的Intent对象

这里展示了如何处理"选择联系人"意图的返回结果：
```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    // Check which request we're responding to
    if (requestCode == PICK_CONTACT_REQUEST) {
        // Make sure the request was successful
        if (resultCode == RESULT_OK) {
            // The user picked a contact.
            // The Intent's data Uri identifies which contact was selected.
            // Do something with the contact here (bigger example below)
        }
    }
}
```

在这个例子中，结果Intent返回了Android中的联系人或者其它People APP提供的Uri内容。

为了可以正确的处理结果，你必须得懂得Intent结果的格式。如果返回结果的Activity结果是你自己Activity的话那非常轻松。Android平台包含的APP提供了它们自己的API，这些API可以让你依靠指定的结果数据。举个例子，People APP(一些老版本上的联系人APP)总是会返回一个URI内容形式的结果。这个URI指明了选择的联系人，Camera APP会返回一个Bitmap对象，这个对象附加在"data"上。

##额外奖励
上面的代码展示了如何获取联系人结果，但是没有详细的解释如何从结果中读取数据，因为这需要有关[content providers](http://android.xsoftlab.net/guide/topics/providers/content-providers.html)的更进一步的讨论。然而，如果你特别好奇，这里有一些代码展示了如何从结果中查询数据，从而在选择的联系人中获得电话号码：
```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    // Check which request it is that we're responding to
    if (requestCode == PICK_CONTACT_REQUEST) {
        // Make sure the request was successful
        if (resultCode == RESULT_OK) {
            // Get the URI that points to the selected contact
            Uri contactUri = data.getData();
            // We only need the NUMBER column, because there will be only one row in the result
            String[] projection = {Phone.NUMBER};
            // Perform the query on the contact to get the NUMBER column
            // We don't need a selection or sort order (there's only one result for the given URI)
            // CAUTION: The query() method should be called from a separate thread to avoid blocking
            // your app's UI thread. (For simplicity of the sample, this code doesn't do that.)
            // Consider using CursorLoader to perform the query.
            Cursor cursor = getContentResolver()
                    .query(contactUri, projection, null, null, null);
            cursor.moveToFirst();
            // Retrieve the phone number from the NUMBER column
            int column = cursor.getColumnIndex(Phone.NUMBER);
            String number = cursor.getString(column);
            // Do something with the phone number...
        }
    }
}
```

> **Note:**在Android 2.3 (API level 9)之前，在[Contacts Provider](http://android.xsoftlab.net/reference/android/provider/ContactsContract.Contacts.html)执行查询需要APP声明[READ_CONTACTS](http://android.xsoftlab.net/reference/android/Manifest.permission.html#READ_CONTACTS)权限。然而，从Android 2.3开始，当选择的结果返回时Contacts/People APP授予了APP一个临时权限来从Contacts Provider读取信息。这个临时的权限值适用于请求的指定联系人，所以不能够查询其它联系人信息，除非你声明了[READ_CONTACTS](http://android.xsoftlab.net/reference/android/Manifest.permission.html#READ_CONTACTS)权限。