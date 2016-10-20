# Lesson03-material
###Το 3ο μάθημα περιλαμβάνει κλήση απομακρυσμένων λειτουργιών (Web Services) με σκοπό την λήψη των προβλέψεων για τις καιρικές συνθήκες απο πραγματικές υπηρεσίες καιρού.

Αρχικά θα πρέπει να εξοικειωθούμε με την κλήση απομακρυσμένων λειτουργιών.
Θα χρησιμοποιήσουμε το http://openweathermap.org/ για να πάρουμε πληροφορίες και προβλέψεις για τον καιρό στην περιοχή μας.
Για να λειτουργήσει χρειάζεται λογαριασμό και ειδικό κλειδί, οπότε καλό είναι να κάνουμε έναν. 
Προσωρινά μπορούμε να καλέσουμε τα Web Services με το API KEY 27949ea6b6dffa1dad1deb925c9b024b που ανήκει στο username *teohaik*
Σημείωση: Επιτρέπονται μόνο 60 κλήσεις ανά λεπτό.
 
Για να πάρουμε τις προβλέψεις για τις επόμενες 7 ημέρες για την πόλη της Θεσσαλονίκης θα πρέπει να καλέσουμε το εξής:

```
api.openweathermap.org/data/2.5/forecast/daily?id=734077&units=metric&cnt=7&APPID=27949ea6b6dffa1dad1deb925c9b024b
```

Τι σημαίνει αυτό το μακαρόνι;
Τι είναι το HTTP Request?    Δες εδώ: http://www.tutorialspoint.com/http/http_requests

Πώς θα καλέσουμε αυτό το Web Service μέσα απο τον κώδικά μας;

Ως προετοιμασία θα πούμε 2 λόγια το logging

![logging types](https://github.com/UomMobileDevelopment/Lesson03-material/blob/master/logging-types.png)


Θα χρειαστούμε αυτό το κομμάτι κώδικα:

```       
            // These two need to be declared outside the try/catch
            // so that they can be closed in the finally block.
            HttpURLConnection urlConnection = null;
            BufferedReader reader = null;

            // Will contain the raw JSON response as a string.
            String forecastJsonStr = null;

            try {
                // Construct the URL for the OpenWeatherMap query
                // Possible parameters are avaiable at OWM's forecast API page, at
                // http://openweathermap.org/API#forecast
                //MODIFIED FOR CITY OF THESSALONIKI, GREECE
                URL url = new URL("http://api.openweathermap.org/data/2.5/forecast/daily?id=734077&mode=json&units=metric&cnt=7");

                // Create the request to OpenWeatherMap, and open the connection
                urlConnection = (HttpURLConnection) url.openConnection();
                urlConnection.setRequestMethod("GET");
                urlConnection.connect();

                // Read the input stream into a String
                InputStream inputStream = urlConnection.getInputStream();
                StringBuffer buffer = new StringBuffer();
                if (inputStream == null) {
                    // Nothing to do.
                    return null;
                }
                reader = new BufferedReader(new InputStreamReader(inputStream));

                String line;
                while ((line = reader.readLine()) != null) {
                    // Since it's JSON, adding a newline isn't necessary (it won't affect parsing)
                    // But it does make debugging a *lot* easier if you print out the completed
                    // buffer for debugging.
                    buffer.append(line + "\n");
                }

                if (buffer.length() == 0) {
                    // Stream was empty.  No point in parsing.
                    return null;
                }
                forecastJsonStr = buffer.toString();
            } catch (IOException e) {
                Log.e("PlaceholderFragment", "Error ", e);
                // If the code didn't successfully get the weather data, there's no point in attemping
                // to parse it.
                return null;
            } finally{
                if (urlConnection != null) {
                    urlConnection.disconnect();
                }
                if (reader != null) {
                    try {
                        reader.close();
                    } catch (final IOException e) {
                        Log.e("PlaceholderFragment", "Error closing stream", e);
                    }
                }
            }

            return rootView;
        }
    }
}

```

μεταφέρουμε τον κώδικα μέσα στη μέθοδο ```onCreateView``` της κλάσης ```PlaceholderFragment``` μετά την κλήση:
```
listView.setAdapter(...)
```

Η εκτέλεση της εφαρμογής σε αυτό το σημείο θα βγάλει σφάλμα και θα κρασάρει την εφαρμογή. 
Το σφάλμα είναι του τύπου ```NetworkOnMainThreadException``` που μας λέει ότι απαγορεύεται να δημιουργούμε συνδέσεις δικτύου στη main και γενικότερα στο MainThread. 

Λίγα λόγια για τα Threads

![Threads](https://github.com/UomMobileDevelopment/Lesson03-material/blob/master/threads.png)


Θα χρειαστούμε μια κλάση η οποία απλοποιεί τη διαδικασία δημιουργίας Thread and UI thread synchronization 

##[AsyncTask](https://developer.android.com/reference/android/os/AsyncTask.html)##

Αφού μελετήσουμε την AsyncTask, ήρθε η ώρα να τη χρησιμοποιήσουμε για να μεταφέρουμε εκεί τον ανωτέρω κώδικα που επικολλήσαμε πρόχειρα μέσα στην PlaceholderFragment.

Ξεκινάμε με εργασίες αναδόμησης:

   1. Rename PlaceholderFragment -> ForecastFragment
   2. Move ForecastFragment to new file
   3. Create a new AsyncTask child called FetchWeatherTask with the networking code snippet from above. Αυτή η κλάση να δηλωθεί ως εσωτερική της ForecastFragment. Η δήλωσή της αρχικά μπορεί να είναι κάπως έτσι:
   ```
   public class FetchWeatherTask extends AsyncTask<Void,Void,Void> {
   ....
   }
   ```
   Μην ξεχάσουμε να υλοποιήσουμε την αφηρημένη μέθοδο ``` protected Void doInBackground(Void... params) {} ```

Τώρα η εφαρμογή τρέχει χωρίς κρασάρισμα αλλά τα δεδομένα μας είναι ακόμη dummy.

Θα προσθέσουμε ένα κουμπί στην κεντρική οθόνη ώστε όταν το πατάμε να φορτώνει τα αληθινά δεδομένα απο την υπηρεσία καιρού. Ωστόσο θα πρέπει να γνωρίζουμε πως αυτή η λύση είναι μόνο για debugging και δεν πρέπει να είναι μόνιμη. Είναι ΚΑΚΗ ιδέα να βασιζόμαστε σε κουμπιά για να ανανεώνουμε τα δεδομένα μας. Δείτε εδώ περισσότερα: https://www.youtube.com/watch?v=VFdIy0GjUEs
Επίσης το TASΚ φόρτωσης δεδομένων απο την υπηρεσία καιρού χρειάζεται βελτίωση καθώς είναι αρκετά "δεμένο" με το UI Activity και οποιαδήποτε αλλαγή στο UI (πχ περιστροφή οθόνης) θα διακόψει το TASK και τη μεταφορά δεδομένων. Για την ώρα όμως παραμένουμε έτσι.

Ας προσθέσουμε το κουμπί ανανέωσης. Θα μπει ως επιλογή στο ΜΕΝΟΥ. Δυο λόγια για το μενού:


![Δομή ΜΕΝΟΥ](https://github.com/UomMobileDevelopment/Lesson03-material/blob/master/menu.png)

Υλοποιούμε τις παρακάτω μεθόδους στην κλάση ForecastFragment

```
@Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setHasOptionsMenu(true);
    }
```
Δηλώνει πως το Fragment αυτό έχει μια δική του επιλογή στο μενού (συγκεκριμένα τη Refresh)
```
    @Override
    public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
        inflater.inflate(R.menu.forecastfragment, menu);
    }
```
κάνει inflate (φορτώνει στο view) την επιλογή Refreash του μενού
```
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        int id = item.getItemId();

       if(id == R.id.action_refresh){
            AsyncTask<Void, Void, Void> weatherTask = new FetchWeatherTask().execute();
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
```
Δηλώνει ποιος κώδικας θα τρέχει όταν πατηθεί η επιλογή Refresh στο μενού. Προσθέσαμε και τη δημιουργία και κλήση του νέου AsyncTask

ΟΜΩΣ! Αν τρέξουμε την εφαρμογή και πατήσουμε Refresh, η εφαρμογή θα κρασάρει και θα σταματήσει! Στα logs βλέπουμε τον λόγο:

```
 Caused by: java.lang.SecurityException: Permission denied (missing INTERNET permission?)
```

Πρέπει να ζητήσουμε την άδεια απο το σύστημα για να πάρουμε πρόσβαση στο Internet. Αυτό γίνεται προσθέτοντας την εντολή

```  
<uses-permission android:name="android.permission.INTERNET" />
```

στο αρχείο AndroidManifest.xml σε αυτό το σημείο:

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.android.sunshine.app" >
   <uses-permission android:name="android.permission.INTERNET" />
....
```

Για να επιβεβαιώσουμε τώρα οτι έρχονται σωστά τα δεδομένα απο την υπηρεσία καιρού προσθέτουμε μια εντολή LOG στο FetchWeatherTask για να εμφανίσουμε τη μεταβλητή forecastJsonStr:

```
 forecastJsonStr = buffer.toString();
 Log.v(LOG_TAG,"Forecast JSON String: "+forecastJsonStr);
```

και βλέπουμε το LOG στην καρτέλα AndroidMonitor του Android Studio. Τα δεδομένα έρχονται κανονικά.


Βελτιώνουμε την δημιουργία του Web Service URL και εισάγουμε παραμετροποίηση στο Fetch Weathet Task έτσι ώστε να δέχεται σαν πρώτη παράμετρο τον κωδικό κάποιας πόλης και να φέρνει τον καιρό απο τη συγκεκριμένη πόλη.

```
 AsyncTask<String, Void, Void> weatherTask = new FetchWeatherTask().execute("734077");
```

Δείτε στο αρχείο ```greek-city-codes.csv``` για κωδικούς απο όλες τις ελληνικές πόλεις.

Το URL θα 'χτιστεί' παραμετρικά με τη βοήθεια του URI Builder:

```
                ......
                // Will contain the raw JSON response as a string.
                String forecastJsonStr = null;
                String weatherFormat = "json";
                String daysForecast = "7";
                String units = "metric";
    
                try {
                    // Construct the URL for the OpenWeatherMap query
                    // Possible parameters are avaiable at OWM's forecast API page, at
                    // http://openweathermap.org/API#forecast
                    //MODIFIED FOR CITY OF THESSALONIKI, GREECE
                    final String baseUrl = "http://api.openweathermap.org/data/2.5/forecast/daily?";
                            //"id=734077&mode=json&units=metric&cnt=7";
                    final String queryParam = "id";
                    final String formatParam= "mode";
                    final String unitsParam= "units";
                    final String daysParam = "cnt";
                    final String apiKeyParam = "APPID";
    
                    Uri builtUri = Uri.parse(baseUrl).buildUpon()
                            .appendQueryParameter(queryParam,params[0])
                            .appendQueryParameter(formatParam,weatherFormat)
                            .appendQueryParameter(unitsParam, units)
                            .appendQueryParameter(daysParam, daysForecast)
                            .appendQueryParameter(apiKeyParam, BuildConfig.OPEN_WEATHER_MAP_API_KEY)
                            .build();
    
                    URL url = new URL(builtUri.toString());
    
                    Log.v(LOG_TAG, "Built URI: "+builtUri.toString());

                // Create the request to OpenWeatherMap, and open the connection
                urlConnection = (HttpURLConnection) url.openConnection();  
                .......
```
