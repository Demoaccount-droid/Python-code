properties([
  parameters([
    choice(
      name: 'AUTOMATIC_SELECTION',
      choices: ['Yes', 'No'],
      description: 'Automatically pick a device?'
    ),

    // Manual device list (only used if AUTO=No)
    activeChoiceParam('MANUAL_DEVICE') {
      description('Pick device manually if AUTO=No')
      choiceType('SINGLE_SELECT')
      groovyScript {
        script("""
          def output = "adb devices".execute().text
          def devices = output.readLines()
                              .findAll { it.contains("\\tdevice") }
                              .collect { it.split()[0] }
          return devices ?: ["No Devices Connected"]
        """)
        fallbackScript('return ["Error getting devices"]')
      }
    ),

    // Final device parameter (auto or manual)
    activeChoiceReactiveParam('FINAL_DEVICE') {
      description('Device chosen automatically or manually')
      choiceType('SINGLE_SELECT')
      referencedParameter('AUTOMATIC_SELECTION,MANUAL_DEVICE')
      groovyScript {
        script("""
          if (AUTOMATIC_SELECTION == "Yes") {
              // Call your lockableResource() method here
              def selected = lockableResource()
              return [selected ?: "No Device Found"]
          } else {
              return [MANUAL_DEVICE]
          }
        """)
        fallbackScript('return ["Device Not Selected"]')
      }
    }
  ])
])
