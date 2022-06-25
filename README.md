### Navigation For ***Compose for Desktop***


#### In order to use this navigation system you have to create 2 files:
1. ***NavController***
2. and ***NavigationHost***

*NavController.kt*
```
import androidx.compose.runtime.Composable
import androidx.compose.runtime.MutableState
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.saveable.rememberSaveable

/**
 * NavController Class
 */
class NavController(
    private val startDestination: String,
    private var backStackScreens: MutableSet<String> = mutableSetOf()
) {
    // Variable to store the state of the current screen
    var currentScreen: MutableState<String> = mutableStateOf(startDestination)

    // Function to handle the navigation between the screen
    fun navigate(route: String) {
        if (route != currentScreen.value) {
            if (backStackScreens.contains(currentScreen.value) && currentScreen.value != startDestination) {
                backStackScreens.remove(currentScreen.value)
            }

            if (route == startDestination) {
                backStackScreens = mutableSetOf()
            } else {
                backStackScreens.add(currentScreen.value)
            }

            currentScreen.value = route
        }
    }

    // Function to handle the back
    fun navigateBack() {
        if (backStackScreens.isNotEmpty()) {
            currentScreen.value = backStackScreens.last()
            backStackScreens.remove(currentScreen.value)
        }
    }
}


/**
 * Composable to remember the state of the navcontroller
 */
@Composable
fun rememberNavController(
    startDestination: String,
    backStackScreens: MutableSet<String> = mutableSetOf()
): MutableState<NavController> = rememberSaveable {
    mutableStateOf(NavController(startDestination, backStackScreens))
}
```


*NavigationHost.kt*
```
import androidx.compose.runtime.Composable

/**
 * NavigationHost class
 */
class NavigationHost(
    val navController: NavController,
    val contents: @Composable NavigationGraphBuilder.() -> Unit
) {

    @Composable
    fun build() {
        NavigationGraphBuilder().renderContents()
    }

    inner class NavigationGraphBuilder(
        val navController: NavController = this@NavigationHost.navController
    ) {
        @Composable
        fun renderContents() {
            this@NavigationHost.contents(this)
        }
    }
}


/**
 * Composable to build the Navigation Host
 */
@Composable
fun NavigationHost.NavigationGraphBuilder.composable(
    route: String,
    content: @Composable () -> Unit
) {
    if (navController.currentScreen.value == route) {
        content()
    }
}
```

#### Create enum class for your screens (You can create your own with more parameters)
```
/**
 * Screens
 */
enum class Screen(
    val label: String,
    val icon: ImageVector
) {
    HomeScreen(
        label = "Home",
        icon = Icons.Filled.Home
    ),
    NotificationsScreen(
        label = "Notifications",
        icon = Icons.Filled.Notifications
    ),
    SettingsScreen(
        label = "Settings",
        icon = Icons.Filled.Settings
    ),
    ProfileScreens(
        label = "User Profile",
        icon = Icons.Filled.VerifiedUser
    )
}
```


##### Now, you are ready to use navigation on ***Compose for Desktop*** app

###### Just create your custom Navigation Host to handle navigation for different screens

```
@Composable
fun CustomNavigationHost(
    navController: NavController
) {
    NavigationHost(navController) {
        composable(Screen.HomeScreen.name) {
            HomeScreen(navController)
        }

        composable(Screen.NotificationsScreen.name) {
            NotificationScreen(navController)
        }

        composable(Screen.SettingsScreen.name) {
            SettingScreen(navController)
        }

        composable(Screen.ProfileScreens.name) {
            ProfileScreen(navController)
        }

    }.build()
}
```

Now you can use it in your ***App*** composable

```
@Composable
fun App() {
    val screens = Screen.values().toList()
    val navController by rememberNavController(Screen.HomeScreen.name)
    val currentScreen by remember {
        navController.currentScreen
    }

    MaterialTheme {
        Surface(
            modifier = Modifier.background(color = MaterialTheme.colors.background)
        ) {
            Box(
                modifier = Modifier.fillMaxSize()
            ) {
                // I have used navigation rail to show how it works
                // You can use your own navbar
                NavigationRail(
                    modifier = Modifier.align(Alignment.CenterStart).fillMaxHeight()
                ) {
                    screens.forEach {
                        NavigationRailItem(
                            selected = currentScreen == it.name,
                            icon = {
                                Icon(
                                    imageVector = it.icon,
                                    contentDescription = it.label
                                )
                            },
                            label = {
                                Text(it.label)
                            },
                            alwaysShowLabel = false,
                            onClick = {
                                navController.navigate(it.name)
                            }
                        )
                    }
                }

                Box(
                    modifier = Modifier.fillMaxHeight()
                ) {

                    // This is how you can use
                    CustomNavigationHost(navController = navController)

                }
            }
        }
    }
}
```

Thanks, for using it.
Don't forget to give a star.
