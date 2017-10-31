# Form Decoration

This ServiceNow application offers added flexibility to the out-of-box form widget in Service Portal, all while retaining it's functionality including respect for form views, UI Policies, and Client Scripts.

Some feature highlights of the app:

- Ability to create your own new field types (e.g. "Address lookup").
	- Apply them to all fields of a certain type, or specific fields on a specific table.
	- Apply to all portals, or a specific portal.
- Put UI Actions anywhere on the page, in any widget, and style them in any way you want.
- Only UI actions, fields, and associated elements are contained in the widget. There's no background colour, or surrounding "frame".

The goal for this application is to be as unopinionated as possible, and allow **you** to control how your form displays.

![Decorated Form](decorated-form-screenshot.png)

## Why?

The out-of-box form widget in Service Portal is extremely powerful, in that it leverages base-platform functionality so that forms can be designed by configuration rather than code.

While it's very powerful, it has it's limitations as well. Specifically, there is no out-of-box way to style elements, or to add special functionality to fields on the form. This means that while a portal can look good, it's appearance is often brought down by the inclusion of the form widget on the page which doesn't conform to the styling of the portal.

Prior to this application, the only way to create your own custom field types and styling was to create your own form from scratch, sacrificing all the base-platform functionality. This app offers a middle-ground.

## Installation

To use this application, simply navigate to `System Applications` > `Studio` on your instance. In the dialog that opens, select `Import From Source Control` and point it at the following repository:

```
https://github.com/platform-experience/form-decoration-app.git
```
**NOTE:** Executing **Apply Remote Changes** from the **Source Control** menu of ServiceNow Studio will uninstall and reinstall the application, dropping all the tables and thus wiping your configuration data, thus it is suggested not to do this. It is aimed for this application to be published to a ServiceNow repo/the store eventually, which will allow you to receive updates to the app without wiping your data.
## Getting Started
There are two widgets included in this application, with the primary one being **Form (Decorated)** (`pi-decorated-form`). This widget must be used for the form decoration functionality to work. For instructions on how to add it to a Service Portal page, refer to the [ServiceNow Documentation website](https://docs.servicenow.com/bundle/helsinki-servicenow-platform/page/build/service-portal/task/t_AddWidgetsToAPage.html).

The application also comes with a Service Portal Page called `df_test` with the widget already on it. If you open the below URL on your instance you can see it in action.

```
/sp?id=df_test&table=sys_user
```

There is another widget called **Form (Decorated) Actions Example** (`pi-df-actions`) which shows an example of how one would use the API to move the UI actions off the form and into another completely different part of the page.

## Understanding

The app works by leveraging the  `template-url` attribute on the out-of-box `spModel` directive. `spModel` is the core directive from which the out-of-box form widget gets it's functionality.

Instead of offering only a finite set of field types (around 20), the template that this app includes offers unlimited types as it's controlled by what is contained in the tables of the application:

- Directive (`x_snc_formdec_directive`) - kind of like a "mini-widget". This is where you create your new field types.
- Decorator (`x_snc_formdec_decorator`) - This is where you set the criteria for when a Directive will be used for a particular form field.

Note that if there is no Decorator defined for a certain field, the out-of-box way of displaying the field will be used.


## Widget Options

The **Form (Decorated)** widget can be controlled with the following options/parameters:

| Option (Label)  | Option (Name)  | URL Parameter | Description | Mandatory | Default |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| Table  | `table` | `table` | The table to show | Yes |
| Sys ID  | `sys_id` | `sys_id` | The Sys ID of the record in the table  | No | `-1` |
| View  | `view` | `view` | The view to use | No | `service_portal` |
| Show only primary UI action  | `primary_only` |  | Whether to only show the primary UI action | No | `false` |
| Show section headings  | `show_headings` |  | Whether to show section headings | No | `true` |
| Show UI actions  | `show_actions` |  | Whether to only show UI actions at all | No | `true` |

If both and option and URL parameter are are supplied, the option will be used.

## Creating a custom field type
To create your own field type, simply create a new record in the **Directive** table (`x_snc_formDec_directive`).

| Field  | Description |
| ------------- | ------------- |
| Name (camel-case)  | The name of the directive in camel-case.  |
| Screenshot  | An example screenshot of the directive.  |
| Description  | A description of the directive, explaining functionality and anything that can't be conveyed in the screenshot.  |
| Template  | HTML template of the field |
| Link function  | Link function of the field |
| Controller function  | Angular controller of the field |
| CSS  | CSS to style the field (not SASS compatible) |

**NOTE:** 

- There are an additional two fields on the table which are read only: `spinal_case` and `snake_case`. These fields will be auto-populated based on the value entered in the **Name (camel-case)** field.
- CSS will be parsed and scoped so it won't conflict with other fields, and the result of this parsing will be stored in the **CSS (parsed)** field (`css_parsed`).

When creating your directive, you need to ensure it interacts with the form correctly so that when the form is submitted, the correct value gets sent to the server.

- The value of your field should in most cases be bound to the `field.stagedValue` scope variable.
- When the user focuses on the field, you should emit the Angular event `sp.spFormField.focus`.
- When the user is no longer interacting with the field, you should emit the Angular event `sp.spFormField.blur` and pass the stagedValue into the FieldValue scope function (e.g. `$scope.fieldValue($scope.field.stagedValue);`

**NOTE:**  be careful when using [one-time bindings](https://toddmotto.com/angular-one-time-binding-syntax/), as if the field/form changes as a result of UI Policy/Client Script you want these updates to reflect on your field.

## Defining when the new field type will be used
You must define a directive record which controlls when the field type will be used. You can do this within the  **Decorators** table (`x_snc_formdec_decorator`).

| Field  | Description |
| ------------- | ------------- |
| Directive  | A reference to the directive record created in step 2. |
| Portal  | The portal where this relationship should apply. |
| Applies to  | Whether the decorator applies to a field type, or a specific field. |
| Field Type | The field for which the directive should be used. |
| Field  | The field for which the directive should be used. |

## Security
The application has 4 roles which can be granted to users to allow them to configure the application.

| Role  | Description | Contains roles |
| ------------- | ------------- | ------------- |
| `x_snc_formDec.admin`  | Administrative access | `x_snc_formDec.developer` `x_snc_formDec.manager ` |
| `x_snc_formDec.developer`  | Can manage directive records | `x_snc_formDec.user` |
| `x_snc_formDec.manager`  | Can manage decorator records | `x_snc_formDec.user` |
| `x_snc_formDec.user`  |  | |

No roles are required to be able to view the form widget or the custom fields.

## Configuration

- [Styling](styling.md)

## Example Directives
The application comes with 9 Directive records in it's demo data which you can use as inspiration for your own custom fields:

- [fdExampleStringField](examples/fdExampleStringField.md)
- fdExampleCheckboxField
- fdExampleChoiceField
- fdExampleEmailField
- fdExamplePasswordField
- fdExamplePhoneField
- fdExamplePhotoField
- fdExampleRefField
- fdExampleYesNoField

There are also 9 Decorator records which will ensure that these field types are used for the `sys_user` table.

## Get assistance

For assistance or feature requests, please raise an issue within this GitHub repo (`platform-experience/form-decoration`). Best efforts will be made to resolve these.