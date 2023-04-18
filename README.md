# Repeat Task Gradle Plugin

Source:
https://stackoverflow.com/questions/75975241/gradle-task-retry-on-failure

## Summary

### Repeat this task

uploadWildFlyWar {
    dependsOn createAndCopySasJar
    description 'Uploads the Wild-fly war'
    group 'publishing'
    repositories {
        mavenDeployer {
            repository(url: repoProps["remoteRepoUrlVal"]) {
                authentication(userName: repoProps["remoteRepoUsernameVal"], password: repoProps["remoteRepoPasswordVal"])
            }
        }
    }
}

### With This

// configure the uploadWildFlyWar task
uploadWildFlyWar { task ->
    // remember the original task actions, copying them to a new list as the  returned list is a live view
    def actions = [*task.actions]
    // replace the original task actions with one own task action that does the error handling
    task.actions = [{
        // remembered exceptions from first two runs
        def exceptions = []
        // try up to three times
        for (i in 1..3) {
            try {
                // execute the original actions
                actions.each { it.execute(task) }
                // if the original actions executed successfully, break the loop
                break
            } catch (e) {
                // on the third try if still throwing an exception
                if (i == 3) {
                    // add the first two exceptions as "suppressed" ones
                    exceptions.each { e.addSuppressed(it) }
                    // rethrow the exception
                    throw e
                } else {
                    // remember the first two exceptions
                    exceptions.add(e)
                    // wait 5 seconds before retrying
                    sleep 5000
                }
            }
        }
    } as Action]
}
