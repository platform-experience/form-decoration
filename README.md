# Form Decoration

This application is designed to easily allow you to style, and add custom functionality to form fields, whilst retaining the power of the out-of-box form widget.

![Decorated Form](decorated-form.png)

**NOTE:** this repository contains documentation only. The repo containing the applicaton can be found at [platform-experience/form-decoration-app](https://github.com/platform-experience/form-decoration-app).

## Background

The out-of-box form widget in Service Portal is extremely powerful, in that it leverages base-platform functionality so that forms can be designed by configuration rather than code. Examples of this functionality are as follows:

- Form views
- Dictionary 
- UI Policies
- Client Scripts
- UI Actions

It also has it's limitations as well. Specifically, there is no out-of-box way to style elements, or to add special functionality (e.g. Google address lookup) to fields/variables on the form. This means that while a portal can look good, it's appearance is often brought down by the inclusion of the form widget on the page which doesn't conform to the styling of the portal.

## Installation

To use this application, simply navigate to `System Applications` > `Studio` on your instance. In the dialog that opens, select `Import From Source Control` and point it at the following repository:

```
https://github.com/platform-experience/form-decoration-app.git
```
**NOTE:** Executing **Apply Remote Changes** from the **Source Control** menu of ServiceNow Studio will uninstall and reinstall the application, dropping all the tables and thus wiping your configuration data, thus it is suggested not to do this. It is aimed for this application to be published to the ServiceNow Store eventually, which will allow you to receive updates to the app without wiping your data.
## Getting Started
There are two primary widgets included in this application, which are self-describing:

1. Decorated Form
2. Decorated Record Producer

These widgets must be used for the form decoration functionality to work. For instructions on how to add these to a Service Portal page, refer to the [ServiceNow Documentation website](https://docs.servicenow.com/bundle/helsinki-servicenow-platform/page/build/service-portal/task/t_AddWidgetsToAPage.html).

There are an additional two widgets to assist with creating new directives, allowing you to inspect the properties of all the fields/variables on the page to ensure your widget is reacting appropriately:

1. Decorated Form (debug)
2. Decorated Record Producer (debug)

### Decorated Form

This widget works similar to the out-of-box form widget, and can be controlled with the following options/parameters:

| Option (Label)  | Option (Name)  | URL Parameter | Description | Mandatory | Default |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| Table  | `table` | `table` | The table to show | Yes |
| Sys ID  | `sys_id` | `sys_id` | The Sys ID of the record in the table  | No | `-1` |
| View  | `view` | `view` | The view to use | No | `service_portal` |
| Show only primary action  | `primary_only` |  | Whether to only show the primary UI action | No | `false` |

In cases above where a URL parameter can be supplied, supplying an option will override a URL parameter if supplied.

## Usage
To create your own custom directive, and use that for a specific field/variable, follow the instruction below:

### 1 - Create an Angular directive
Create a new UI Script and add your directive in it, ensuring it is added to the `formDec` Angular module. For example:

```
angular.module('formDec').directive('myDirectiveName', function () {
	'use strict';
	
	var template = 'YOUR-HTML-TEMPLATE';
	
	var link = function (scope, elem, attrs, ctrl) {
	
		scope.inputField = elem.find('input');
		scope.inputField.on('focus', scope.focusHandler).on('blur', scope.blurHandler);
		
		$timeout(function () {
			scope.$emit("sp.spFormField.rendered", elem, scope.inputField);
		});
	}
	
	return {
		template: template,
		link: link,
		controller: 'fdFieldCtrl',
		require: ['^spModel', '^fdFormField']
	};
});
```

You can add as many directives into this UI Script as you like.

Some key points to be aware of when creating your directive:

- Your directive must use the `fdFieldCtrl` controller, which includes a few helper functions that you will need to call in your link function.
- Your directive will have access to a number of scope variables which will automatically update when the field/form changes (e.g. as a result of UI Policy or Client Script).
	- `field`
		- `name`
		- `directive`
		- `value`
		- `stagedValue`
		- `isReadOnly()`
		- `isMandatory()`
		- `max_length`
		- `visible`
		- `choices`
	- `formModel`:
- Be careful when using [one-time bindings](https://toddmotto.com/angular-one-time-binding-syntax/), as if the field/form changes as a result of UI Policy/Client Script you want these updates to reflect on your field.
- The value of your field should in most cases be bound to the `field.stagedValue` scope variable.
- When the user focuses on the field, you should call the function `focusHandler()`.
- When the user is no longer interacting with the field, you must call the scope function `blurHandler()`. This will do a few things, and importantly will copy the value from the `field.stagedValue` into `field.value` (`field.value` is what gets sent to the server on submit).


To see an example of the above, please refer to the UI Script included in the application `x_snc_formdec.exampleDirectives`. This includes 4 directives covering different types of fields.

Once you've created your UI Script, create an entry in the `sp_js_include` table which points to it, and then create a record in the `m2m_sp_dependency_js_include` table linking the JS Include you created to the **Form Decoration** Widget Dependency.

### 2 - Declare the directive's existence

Once you've created the actual Angular directive, you need to define it in ServiceNow within the **Directives** table (`x_snc_formdec_directives`).

| Field  | Description |
| ------------- | ------------- |
| Name (camel-case)  | The name of the directive in camel-case.  |
| Screenshot  | An example screenshot of the directive.  |
| Description  | A description of the directive, explaining functionality and anything that can't be conveyed in the screenshot.  |

**NOTE:** There are an additional two fields on the table which are read only: `spinal_case` and `snake_case`. These fields will be auto-populated based on the value entered in the **Name (camel-case)** field.

### 3 - Define the directive to field/variable relationship
You must define a relationship between the directive you created and the form field/variable. Doing this will mean that when the field/variable is shown on the form, the directive will be used to display it instead of the out-of-box way of representing it. 

You can define the relationship using the **Decorators** table (`x_snc_formdec_decorators`).


| Field  | Description |
| ------------- | ------------- |
| Directive  | A reference to the directive record created in step 2. |
| Portal  | The portal where this relationship should apply. |
| Field  | The field for which the directive should be used. |
| Variable  | The variable for which the directive should be used. |

**NOTE:** you can link a single **Decorator** record to both a field AND variable if desired, however it's probably better to have a Decorator record for each.

## Configuration

### Styling
The following SASS variables can be set to control the styling of the form itself.

| Variable  | Effect | Default |
| ------------- | ------------- | ------------- |
| `$fd-ff-margin`  | Margin between each form field| `10px 10px 10px 10px` |
| `$fd-ui-actions-margin` | Margin surrounding the UI Actions | `30px 0px` |
| `$fd-ui-actions-text-align` | Text-alignment of the UI Actions | `center` |
| `$fd-uia-margin` | Margin between individual UI Actions | `10px` |
| `$fd-uia-border-radius` | Border radius of the UI Actions | `8px` |
| `$fd-uia-border` | Border of the UI Actions | `3px solid #C4D0D6` |
| `$fd-uia-background` | Background of the UI Actions | `#fff` |
| `$fd-uia-padding` | Padding of the UI Actions | `5px 20px` |
| `$fd-uia-color` | Color of the UI Actions | `#000` |
| `$fd-uia-min-height` | Minimum height of the UI Actions | `40px` |
| `$fd-uia-min-width` | Minimum width of the UI Actions | `40px` |
| `$fd-uia-font-size` | Font size of the UI Actions | `18px` |
| `$fd-uia-background-hover` | Background of UI Action when hovered | `#F9FDFF` |
| `$fd-uia-background-border` | Border of UI Action when hovered | `#F9FDFF` |
| `$fd-uia-background-color` | Color of UI Action when hovered | `#F9FDFF` |
| `$fd-uia-primary-background` | Background of the primary UI Action | `#da1a40` |
| `$fd-uia-primary-border` | Border of the primary UI Action | `#da1a40` |
| `$fd-uia-primary-color` | Color of the primary UI Action | `#fff` |
| `$fd-uia-primary-background-hover` | Background of the primary UI Action when hovered | `#fd3056` |
| `$fd-uia-primary-border-hover` | Border of the primary UI Action when hovered | `#da1a40` |
| `$fd-uia-primary-color-hover` | Color of the primary UI Action when hovered | `#fff` |

## Get assistance

For assistance or feature requests, please raise an issue within this GitHub repo (`platform-experience/form-decoration`). Best efforts will be made to resolve these.