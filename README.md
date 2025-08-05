properties([
  parameters([
    choice(name: 'AUTO_DEVICE_PICK', choices: ['Yes', 'No'], description: 'Pick device automatically?'),

    [$class: 'CascadeChoiceParameter',
      choiceType: 'PT_SINGLE_SELECT',
      filterable: false,
      name: 'SELECTED_DEVICE',
      randomName: 'choice-device-uid',
      referencedParameters: 'AUTO_DEVICE_PICK',
      script: [
        $class: 'GroovyScript',
        fallbackScript: [classpath: [], sandbox: false, script: 'return ["No Devices"]'],
        script: [classpath: [], sandbox: false, script: '''
          if (AUTO_DEVICE_PICK == "Yes") {
              def devices = "adb devices".execute().text.readLines()
                            .findAll { it ==~ /[a-zA-Z0-9]+\\s+device/ }
                            .collect { it.tokenize()[0] }
              return devices ? [devices[0]] : ["No Devices Found"]
          } else {
              def devices = "adb devices".execute().text.readLines()
                            .findAll { it ==~ /[a-zA-Z0-9]+\\s+device/ }
                            .collect { it.tokenize()[0] }
              return devices ?: ["No Devices Found"]
          }
        ''']
      ]
    ]
  ])
])
