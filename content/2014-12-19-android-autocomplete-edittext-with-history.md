---
title: "Android AutoComplete EditText with history"
author: "Nicklas Møller Jepsen"
comments: "true"
date: "2014-12-19"
url: "/2014/12/19/android-autocomplete-edittext-with-history"
comments: "true"
categories:
  - android
  - programming
tags:
  - history
  - autocomplete
  - recent
  - editText
  - beginner
series:
---
## Introduction
In this post I'm going to show you have to get an Android app to have some autocomplete/history functionality.
You can achieve this by using the AutoCompleteEditText.<!--more-->

If you havent already set up Android Studio, then [see this post]() where I guide you on installing Android Studio and getting completely set up for Android development!

## The Auto Complete EditText
We will use the AutoCompleteEditText which is made just for this purpose.

However, we need a place to store the user's input and for that we are using SharedPreferences.

It will be wrapped in a Activity, ready for you to try out!

### Implementation
First we're going to declare the view in the actovity_main.xml add the following:
{{< highlight xml >}}

<AutoCompleteTextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:id="@+id/textInput"
    android:layout_marginTop="108dp"
    android:layout_alignParentTop="true"
    android:layout_alignParentLeft="true"
    android:layout_alignParentRight="true"
    android:editable="true"
    android:hint="Enter something"/>

{{< /highlight >}}

The go to your MainActivity.java and we're going to setup the methods and variables we need to store the history in the SharedPreferences.
Create a couple of class variables:

{{< highlight  java >}}

public static final String PREFS_NAME = "PingBusPrefs";
public static final String PREFS_SEARCH_HISTORY = "SearchHistory";
private SharedPreferences settings;
private Set<String> history;

{{< /highlight >}}
These should explain them selves; it is just some strings used as keys for the preferences and the the settings variable and a set to contain the actual data of the history/input from the user.

Now we add a method to load create an adapter used by our AutoCompletEditText to show recent history:
{{< highlight java >}}

private void setAutoCompleteSource()
{
    AutoCompleteTextView textView = (AutoCompleteTextView) findViewById(R.id.textInput);
    ArrayAdapter<String> adapter = new ArrayAdapter<String>(
		this, android.R.layout.simple_list_item_1, history.toArray(new String[history.size()]));
    textView.setAdapter(adapter);
}

{{< /highlight >}}
Then we add a method we can call to add a history item:
{{< highlight java >}}

private void addSearchInput(String input)
{
    if (!history.contains(input))
    {
        history.add(input);
        setAutoCompleteSource();
    }
}

{{< /highlight >}}

Last but not least, we're going to implement the save method, so that we can save the preferences:

{{< highlight java >}}

private void savePrefs()
{
    SharedPreferences settings = getSharedPreferences(PREFS_NAME, 0);
    SharedPreferences.Editor editor = settings.edit();
    editor.putStringSet(PREFS_SEARCH_HISTORY, history);

    editor.commit();
}

{{< /highlight >}}

Now that we have these methods, we're going to initialize everything in the onCreate method. First add these 3 lines:

{{< highlight java >}}

settings = getSharedPreferences(PREFS_NAME, 0);
history = settings.getStringSet(PREFS_SEARCH_HISTORY, new HashSet<String>());

setAutoCompleteSource();

{{< /highlight >}}
This will initilize the settings and read the history from the settings.
Then we're going to wire up the onKey event on the AutoCompleteEditText and write the implementation to to listen for "Enter" key presses:

{{< highlight java >}}

// Set the "Enter" event on the search input
final AutoCompleteTextView textView = (AutoCompleteTextView) findViewById(R.id.textInput);
textView.setOnKeyListener(new View.OnKeyListener()
{
    public boolean onKey(View v, int keyCode, KeyEvent event) {
        // If the event is a key-down event on the "enter" button
        if ((event.getAction() == KeyEvent.ACTION_DOWN) &&
                (keyCode == KeyEvent.KEYCODE_ENTER)) {

            addSearchInput(textView.getText().toString());
            return true;
        }
        return false;
    }
});

{{< /highlight >}}

Now the last thing we need, is to call the savePrefs method in the onStop method which will then make sure that anychanges made to the history will be copied to the settings and the settings will be save:

{{< highlight java >}}

@Override
protected void onStop()
{
    super.onStop();

    savePrefs();
}

{{< /highlight >}}