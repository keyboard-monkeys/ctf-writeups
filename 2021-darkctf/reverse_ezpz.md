This is an android challenge and I used my own device to run the provided APK (probably not a good idea). However, it should be doable with an emulator as well.

Decompiling the apk online and searching through it, `com.application.ezpz.MainActivity` contains the lines

```java
final String[] YEET = new whyAmIHere().isThisWhatUWant();

...
...

} else if (YEET[0].equals(MainActivity.this.button.getText().toString())) {
    Toast.makeText(MainActivity.this.getApplicationContext(), "Well thats the  Correct Flag", 0).show();
```

The source for `com.application.ezpz.whyAmIHere.java` shows how `YEET` is created and populated:

```java
public class whyAmIHere {
    public String[] isThisWhatUWant() {
        final String[] justAWaytoMakeAsynctoSync = {""};
        FirebaseFirestore.getInstance().collection("A_Collection_Is_A_Set_Of_Data").get().addOnSuccessListener(new OnSuccessListener<QuerySnapshot>() {
            public void onSuccess(QuerySnapshot queryDocumentSnapshots) {
                Iterator<QueryDocumentSnapshot> it = queryDocumentSnapshots.iterator();
                while (it.hasNext()) {
                    justAWaytoMakeAsynctoSync[0] = it.next().getString("Points");
                    Log.d("TypicalLogcat", justAWaytoMakeAsynctoSync[0]);
                }
            }
        }).addOnFailureListener(new OnFailureListener() {
            public void onFailure(Exception e) {
                justAWaytoMakeAsynctoSync[0] = "Something Failed,Maybe Contact Author?";
            }
        });
        return justAWaytoMakeAsynctoSync;
    }
}
```

As `YEET` is loaded immediately after the first time `editText` is clicked, it should be in memory shortly after clicking `editText`. Debugging the apk in android studio, a heap dump can be generated and exported after the box has been clicked. Then, strings from the heap dump can be searched for the wrapper `darkCON`, which gives `darkCON{d3bug_m5g_1n_pr0duct10n_1s_b4d}`.
