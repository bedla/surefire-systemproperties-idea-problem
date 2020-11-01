Repo to reproduce a problem with IDEA's JUnit run configuration with conjunction of Maven surefire plugin inside specific Maven's profile.

Issued https://youtrack.jetbrains.com/issue/IDEA-254320

## Steps to reproduce

- Open this project.
- Click refresh Maven project button.
- Run `XxxTest` and expect test fail
- Now check and run same test with `profile1` Profile.
- Results is also failure, but success is expected.

## Workaround

- After `profile1` is checked, click refresh Maven project button again.
- Run `XxxTest` and see all green result

## Tech info

- IDEA has it's own mirror model of Maven project in project's sta
- When uses Remove Maven server to apply profiles, see https://github.com/JetBrains/intellij-community/blob/idea/202.7660.26/plugins/maven/src/main/java/org/jetbrains/idea/maven/project/MavenProjectReader.java#L380
  - it serializes Maven project model, sends it using RMI, deserializes it and map to Maven's DTOs
- This model is not full model but portion of it.
- Later, result from Remove Maven server, is used to update internal `myState` of `MavenProject`
- Method `doSetResolvedAttributes` is used to update/merge this internal state, see https://github.com/JetBrains/intellij-community/blob/idea/202.7660.26/plugins/maven/src/main/java/org/jetbrains/idea/maven/project/MavenProject.java#L184
- Naive implementation send to remote `applyProfiles` methods also definition of plugins from profiles.
  - but new plugins resolved from profiles are added to list of plugins
  - this creates duplice items
  - see https://github.com/bedla/intellij-community/commit/2a81f1b7de9c16f06a5d8b855f367d7badb4501e
- Later when JUnit run configuration is invoked `MavenJUnitPatcher` is used.
  - it patches run command line with values from `systemPropertyVariables` defined as configuration of `maven-surefire-plugin`
  - for details see https://github.com/JetBrains/intellij-community/blob/idea/202.7660.26/plugins/maven/src/main/java/org/jetbrains/idea/maven/execution/MavenJUnitPatcher.java#L55
    