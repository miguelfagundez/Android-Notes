# Security Notes
Some notes about basic security in android development

## EncryptedFile & EncryptedSharedPreferences

**Reference**: `androidx.security.crypto`

Complete docs [here](https://developer.android.com/reference/androidx/security/crypto/package-summary)

These classes are used to create and read encrypted files. Using *EncryptedFile* and *EncryptedSharedPreferences* allows you to locally protect files that may contain sensitive data, API keys, OAuth tokens, and other types of secrets.

### Read / Write files using `EncryptedFile`

Similar to File, `EncryptedFile` provides a *FileInputStream* object for reading and a *FileOutputStream* object for writing. Files are encrypted using Streaming AEAD, which follows the OAE2 definition. The data is divided into chunks and encrypted using `AES256-GCM` in such a way that it's not possible to reorder.

First, we need to generate a symetric key using `MasterKey.Builder`, and this key is stored in the `Android Keystore system`.

```kotlin
 val masterKey = MasterKey.Builder(context)
                  .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
                  .build()
```

Second, we create the file and use the MasterKey.

```kotlin
    val file = File(context.filesDir, "secret_data")

    val encryptedFile = EncryptedFile.Builder(
        context,
        file,
        masterKey,
        EncryptedFile.FileEncryptionScheme.AES256_GCM_HKDF_4KB
    ).build()
```

Finally, we can write or read our file.

```kotlin
// Checking if file exist before write
if (fileToWrite.exists()) {
  fileToWrite.delete()
}
// write to the encrypted file
 val encryptedOutputStream = encryptedFile.openFileOutput().apply{
  ...
 }

 // read the encrypted file
 val encryptedInputStream = encryptedFile.openFileInput().read()
```


### Read / Write files using `EncryptedSharedPreferences`

This is an implementation of `SharedPreferences` that encrypts keys and values. If your application needs to save Key-value pairs - such as API keys - Android provides the *EncryptedSharedPreferences* class, which uses the same SharedPreferences interface that you’re used to.

Both keys and values are encrypted. Keys are encrypted using [AES256-SIV-CMAC](https://datatracker.ietf.org/doc/html/rfc5297), which provides a deterministic cipher text; values are encrypted with `AES256-GCM` and are bound to the encrypted key. This scheme allows the key data to be encrypted safely, while still allowing lookups.

**Reference**: `androidx.security.crypto`

Complete docs [here](https://developer.android.com/reference/androidx/security/crypto/EncryptedSharedPreferences)

First, we need to generate a symetric key using `MasterKey.Builder`, and this key is stored in the `Android Keystore system`.

```kotlin
val masterKey = new MasterKey.Builder(context)
     .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
     .build()
```

Second, we create the shared preferences file.

```kotlin
val sharePreferences = EncryptedSharedPreferences.create(
        context,
        "Share_preferences",
        masterKey,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )
```

Finally, we can write or read the file as we normally would.

```kotlin
// writing
sharePreferences.edit().apply {
        putBoolean("KEY", false)
        apply()
        Log.d(TAG, "Encrypted SharePreferences - Written successfully")
    }

// reading
val value = sharePreferences.getBoolean("KEY", false)
```

[FileLocker](https://github.com/android/security-samples/tree/master/FileLocker) is a sample app on the Android Security GitHub samples page. It’s a great example of how to use File encryption using Jetpack Security.
