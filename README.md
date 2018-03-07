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

# Activar MultiDex 
1. Agregar libreria:
```gradle
implementation 'com.android.support:multidex:1.0.3'
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