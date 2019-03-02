# Nuevo Proyecto en Android
Aqui encontraras los pasos recomendados para crear un proyecto en Android.

1. Inicia Android Studio, File -> New Project
2. Asignamos Nombre, Paquete y localización del mismo.
3. Seleccionar version minima de compilación.
4. Seleccionar si es necesario ya cree un template base.
5. Iniciar GIT: VCS -> Enable Version Control Integration -> Git
6. Comitiar los primeros archivos: VCS -> Commit Changes (Aqui seleccionar los archivos necesarios)

# Desactivar ActionBar y Activar la nueva Toolbar
1. Abrir styles.xml
2. Agregar dentro de AppTheme:
```xml
<!-- Habilitar la Toolbar -->
<item name="windowActionBar">false</item>
<item name="windowNoTitle">true</item>
```

# Crear Clase para las Constantes:
1. Crear archivo "Constant", aqui se pueden guardar las KEYs de las APIs, y otros datos importantes que pueden variar dependiendo el entorno.
```kotlin
class Constant {
    
    companion object {
        @JvmField
        val MOBILEIA_APP_ID = 52423
    }
}
```

# agregar libreria base MobileIA
1. Agregar dependencias:
```gradle
implementation 'com.mobileia:core:0.0.26'
```

# Activar MultiDex ()
1. Agregar libreria:
```gradle
implementation 'com.android.support:multidex:1.0.3'

implementation 'androidx.multidex:multidex:2.0.1' (AndroidX)
```
2. Crear archivo MainApplication, que herede de MultiDexApplication:
```java
public class MainApplication extends MultiDexApplication {

    @Override
    public void onCreate() {
        super.onCreate();
        // Configurar Mobileia Lab
        Mobileia.getInstance().setAppId(Constant.MOBILEIA_LAB_APP_ID);
    }
}
```
```kotlin
class MainApplication: MultiDexApplication() {

    override fun onCreate() {
        super.onCreate()
        // Configuramos MobileIA Lab
        Mobileia.getInstance().appId = Constant.MOBILEIA_APP_ID
    }
}
```
3. Editar el Manifest y para agregar el application:
```xml
<application
        android:name=".MainApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
```
4. Editamos el build.gradle:
```gradle
android {
    compileSdkVersion 27
    buildToolsVersion '27.0.2'
    defaultConfig {
        applicationId "com.mobileia.prode"
        minSdkVersion 16
        targetSdkVersion 27
        versionCode 1
        versionName "0.0.1"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
        vectorDrawables.useSupportLibrary = true

        // Enabling multidex support.
        multiDexEnabled true

    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    // Enabling multidex support.
    dexOptions {
        javaMaxHeapSize "4g"
    }


}
```

# Utilizando Realm
1. Crear entidad:
```kotlin
@RealmClass
class HashItem : RealmObject() {

    @PrimaryKey
    var id: Int = 0
    var category_id: Int = 0
    var title: String? = ""
    var caption: String? = ""
    var language: Int = 0
}

@RealmClass
class HashCategory : RealmObject() {

    @PrimaryKey
    var id: Int = 0
    var title: String? = ""
    var image: String? = ""
    var items: RealmList<HashItem> = RealmList<HashItem>()
}

```

# Utilizando Retrofit & RXJava
1. Crear archivo base para todos los Servicios:
```kotlin
class BaseRest : MobileiaRest() {

    fun userService(): UserService {
        return createService(UserService::class.java)
    }

    override fun getBaseUrl(): String {
        return Constant.BASE_API_URL
    }

}
```
2. Creacion de un Servicio donde se agregan los diferentes endpoints de la API:
```kotlin
interface UserService {

    @FormUrlEncoded
    @POST("user/add")
    fun createUser(@Field("email") email: String, @Field("password") password: String): Observable<User>
}
```
3. Usar API:
```kotlin
BaseRest().userService().createUser("mimail@mobileia.com", "123456")
            .subscribeOn(Schedulers.newThread())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe { response ->

                if(response.success){
                    txtMain.text = "Respuesta Correcta - : " + response.id
                }else {
                    txtMain.text = "Respuesta Incorrecta"
                }

            }
```


# Creación de un modulo publico para integrar con Gradle:
1. File -> New Module
2. Seleccionar Android Library
3. Configuramos nombre y paquete. (Tener en cuenta que el nombre del modulo, debe ser el que va a figurar en Gradle).
4. Agregado en el "build.gradle" del proyeto, los paquetes requeridos para poder subir la libreria a JCenter:
```gradle
buildscript {
    
    repositories {
        google()
        jcenter()
    }
    dependencies {

        // ...

        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.7.3'
        classpath 'com.github.dcendents:android-maven-gradle-plugin:1.4.1'

        // ...
    }
}
```
5. Copiamos archivos "installv1.gradle" y "bintrayv1.gradle" en la carpeta raiz del proyecto.
6. Abrimos archivo "local.properties" para agregar API_KEY de bintray para tener permisos para subir la libreria:
```gradle
bintray.user=mobileia
bintray.apikey=23nasjhdl4daslcl67fjs3wef%Fsdcl5w4
```
7. Agregamos los datos de la libreria en el "build.gradle" del modulo, editar con los datos de tu libreria (Tener en cuenta que el artifact debe ser el mismo que el nombre del modulo):
```gradle
apply plugin: 'com.android.library'

ext {
    bintrayRepo = 'maven'
    bintrayName = 'mobileia-twitter'

    publishedGroupId = 'com.mobileia'
    libraryName = 'MIA Twitter'
    artifact = 'twitter'

    libraryDescription = 'Library for Android.'

    siteUrl = 'https://github.com/MobileIA/mia-twitter-android'
    gitUrl = 'https://github.com/MobileIA/mia-twitter-android.git'

    libraryVersion = '0.0.2'

    developerId = 'mobileia'
    developerName = 'MobileIA'
    developerEmail = 'matias.camiletti@mobileia.com'

    licenseName = 'The Apache Software License, Version 2.0'
    licenseUrl = 'http://www.apache.org/licenses/LICENSE-2.0.txt'
    allLicenses = ["Apache-2.0"]
}
```
8. Agregamos en el mismo archivo anterior al final lo siguiente:
```gradle
apply from: '../installv1.gradle'
apply from: '../bintrayv1.gradle'
```
9. Por ultimo abrimos la terminal desde Android Studio y ejecutamos los siguiente comandos:
```console
./gradlew install
./gradlew bintrayUpload
```
10. Si todo fue correcto deberia mostrarse en la consola: "BUILD SUCCESSFUL"
11. Si tienes problemas al subir una libreria con archivos Kotlin, agregar siguiente Linea en el Build.gradle del Grupo:
```gradle
allprojects {
    ...

    tasks.withType(Javadoc).all { enabled = false }
}
```

# Librerias Utiles/Recomendadas:
* MobileIA Core: https://github.com/MobileIA/mia-core-android
* MobileIA Helpers: https://github.com/MobileIA/helper-android (Deprecated)