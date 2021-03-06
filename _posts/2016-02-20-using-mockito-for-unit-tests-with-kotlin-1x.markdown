---
title: Using Mockito for unit testing with Kotlin (1/x)
layout: post
categories: Android
---

Dependencies:

``` groovy

// Android stuff...

dependencies {
    //...
    compile "org.jetbrains.kotlin:kotlin-stdlib:1.0.0"

    testCompile 'junit:junit:4.12'
    testCompile 'org.mockito:mockito-core:2.0.42-beta'
    testCompile('com.squareup.assertj:assertj-android:1.1.1') {
        exclude group: 'com.android.support', module: 'support-annotations'
    }
}

```

# Little Notes
* All the functions AND PROPERTIES should be open, by default functions and properties are final and mockito cant mock them.
* If you want to use `spy` you should use [mockito-kt](https://github.com/paslavsky/mockito-kt).

# Example

We will show:

1. `Settings`: manage settings storage, where to store them and which ones.
1. `SettingsView`: ...
1. `SettingsPresenter`: manage the bussiness logic for the settings.

You may have a class `Settings`: 

```

open class Settings(context: Context) {
    val localCache = LocalCache(context)

    open var playJustWithHeadphones: Boolean
        get() = localCache.get("headphones", false)
        set(value) = localCache.put("headphones", value)
}

```

Then a presenter that use those settings:

```

class SettingsPresenter {
    private var mSettings: Settings? = null
    private var mView: SettingsView? = null

    fun onCreate(view: SettingsView, settings: Settings) {
        mView = view
        mSettings = settings

        view.setHeadphonesToggleCheck(settings.playJustWithHeadphones)
    }
}

```


Check that mocked Settings class **is open** and mocked var property **is open**

Then the tests passing:

```

class SettingsPresenterTests {
    @Mock lateinit var mockedView: SettingsView
    @Mock lateinit var mockedSettings: Settings
    lateinit var presenter: SettingsPresenter

    @Before
    fun setUp() {
        MockitoAnnotations.initMocks(this)
        presenter = SettingsPresenter()
    }

    @Test
    fun test_onCreate_updateGui() {
        Mockito.`when`(mockedSettings.playJustWithHeadphones).thenReturn(true)
        presenter.onCreate(mockedView, mockedSettings)

        Mockito.verify(mockedView).setHeadphonesToggleCheck(true)
    }
}

```

# Notes
- Using `lateinit` to let the variables be initialized on `@Before` and avoid using `?` or `!!` all over the tests.
- `SettingsView` and `Settings` are mocked using `MockitoAnnotations`