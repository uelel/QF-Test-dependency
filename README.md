# QF-Test dependency

This is a basic implementation of dependency mechanism in `QF-Test`.  
A dependency allows testcases to start running from defined SUT screens (states).

In this example, two desired states `loginScreen` and `registrationScreen` are defined. With proper resolve of SUT states, `QF-Test` is able to get to either login screen/registration screen and ensure proper execution of following testcases.

## How it works
The procedure tries to iteratively resolve SUT screen based on defined state:action pairs.

**In this example, two actions are pursued during each iteration:**  
1. Action based on current component (`dependency.executeActionForMessageBox`)  
2. Action based on current screen title (`dependency.executeActionForTitle`)

Error is logged in case desired state could not be achieved during the iteration process:

```python
from time import sleep
import re

desiredState = rc.lookup("desiredState")

# Get to desired state
for i in range(5):
    print '\nGetting to %s screen %i/5:' % (desiredState, i+1) 

    # Check whether message box is displayed and execute corresponding action
    status = rc.callProcedure("dependency.executeActionForMessageBox")

    # Get current title
    title = rc.callProcedure("label.gettitle")
    print 'Current title: %s' % title

    # Execute action for given title
    status = rc.callProcedure("dependency.executeActionForTitle", {"title": title, "desiredState": desiredState})
    
    # Break loop in case desired state was achieved
    if status == 'finished':
      break
    
    sleep(1.0)
    if i == 4: rc.logError('dependency.getToDesiredScreen: State could not be resolved!')
```

## 1. Action based on current component

Actions are procedures that are called when a message box component is found on the screen.  
Actions are defined in Python dictionary with format compatible to `rc.callProcedure`:  
`<component_id>: {"proc": <procedure_name>, "pars": <parameters_dictionary>}`

```python
global messageActionDict

# Define component names of message icons and corresponding actions
messageActionDict = {
  'labelerrormessageicon': {'proc': 'button.press', 'pars': {'label': rc.lookup("btnNext"), 'logError': False}}, # error message box
  'labelerrormessageicon2': {'proc': 'button.press', 'pars': {'label': rc.lookup("btnNext"), 'logError': False}}, # error message box
  'panelmessageicon': {'proc': 'button.press', 'pars': {'label': rc.lookup("btnYes"), 'logError': False}} # question message box
}
```

## 2. Action based on current screen title

Actions are procedures that are called when corresponding title is found on the screen.  
Actions are defined in Python dictionary with format compatible to `rc.callProcedure`:  
`<title>: {"proc": <procedure_name>, "pars": <parameters_dictionary>}`

```python
# Define titles and corresponding actions
global stateActionDict
stateActionDict = {
    rc.lookup("mask_login"): {'proc': 'dependency.resolveLogin', 'pars': {'desiredState': rc.lookup("desiredState")}},
    rc.lookup("mask_registration"): {'proc': 'dependency.resolveRegistration', 'pars': {'desiredState': rc.lookup("desiredState")}},
    rc.lookup("mask_logout"): {'proc': 'dependency.completeLogout', 'pars': {'user': rc.lookup("user"), 'pwd': rc.lookup("pwd")}},
    rc.lookup("mask_break"):  {'proc': 'dependency.completeLogout', 'pars': {'user': rc.lookup("user"), 'pwd': rc.lookup("pwd")}},
    rc.lookup("mask_selectOption"): {'proc': 'button.press', 'pars': {'label': rc.lookup("btnCancel"), 'logError': False}}
}
```
