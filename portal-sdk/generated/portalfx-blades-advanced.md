
<a name="advanced-topics"></a>
## Advanced Topics

The following sections discuss advanced topics in template blade development.

* [Deep linking](#deep-linking)

* [Displaying notifications](#displaying-notifications)

* [Pinning the blade](#pinning-the-blade)

* [Storing settings](#storing-settings)

* [Displaying Unauthorized UI](#displaying-unauthorized-ui)

* [Dynamically displaying Notice UI](#dynamically-displaying-notice-ui)

**NOTE**: In this discussion, `<dir>` is the `SamplesExtension\Extension\` directory, and  `<dirParent>`  is the `SamplesExtension\` directory, based on where the samples were installed when the developer set up the SDK. If there is a working copy of the sample in the Dogfood environment, it is also included.

* * *

<a name="advanced-topics-deep-linking"></a>
### Deep linking

Deep linking is the feature that displays a URL that directly navigates to the new blade when a parent blade is opened and the Portal URL is updated. By design, only certain blades can be deep linked.

Blades that cannot be deep linked are the ones that cannot be opened independent of a parent blade or part, like blades that return values to a calling module. An example of blades that cannot be deep-linked is a Web page in the middle of a website's check-out experience.

One of the easiest ways to make a deep-linkable blade is to mark its  TemplateBlade as pinnable. For more information about pinning blades, see [#pinning-the-blade](#pinning-the-blade).

<a name="advanced-topics-displaying-notifications"></a>
### Displaying notifications

A status bar can be displayed at the top of a blade that contains both text and coloration that convey informations and status to users. For example, when validation fails in a form, a red bar with a message is displayed at the top of the blade. This area is clickable and can open a new blade or an external URL.

This capability is exposed by adding the `statusBar` member to the Blade base class. Use `this.statusBar(myStatus)` in the `ViewModel`, as in the code located at `<dir>Client/V1/Blades/ContentState/ViewModels/ContentStateViewModels.ts`.
It is also included in the following code.

```typescript

if (newContentState !== MsPortalFx.ViewModels.ContentState.None) {
    statusBar = {
        text: newDisplayText,
        state: newContentState,
        selection: stateDetailsBladeSelection,
        onActivated: onActivated
    }
}

this.statusBar(statusBar);

```

<a name="advanced-topics-pinning-the-blade"></a>
### Pinning the blade

Blades can be marked as pinnable to the dashboard by setting `Pinnable="true"` in the TemplateBlade's PDL definition file. Blades are pinned as button parts to the dashboard by default. Any other represention should be specified in the PDL file. 

<a name="advanced-topics-storing-settings"></a>
### Storing settings

Settings that are associated with a blade can be stored. Those settings need to be declared both in the PDL definition file and in the `ViewMmodel` for the blade.  The code that demonstrates how to store settings is located at  `<dir>Client/V1/Blades/Template/Template.pdl` and  `<dir>Client/V1/Blades/Template/ViewModels/TemplateBladeViewModels.ts`.

The process is as follows.

Specify the settings in the PDL file using the `TemplateBlade.Settings` element.

```xml

<TemplateBlade Name="PdlTemplateBladeWithSettings"
               ViewModel="{ViewModel Name=TemplateBladeWithSettingsViewModel, Module=./Template/ViewModels/TemplateBladeViewModels}"
               Template="{Html Source='Templates\\TemplateBladeWithSettings.html'}">
  <TemplateBlade.Settings>
    <Setting Property="colorSettingValue" />
    <Setting Property="fontSettingValue" />
  </TemplateBlade.Settings>
</TemplateBlade>

```

After the settings are declared, they should also be specified in the ViewModel, as in the following example.

<!-- TODO:  Determine why the samples in this section are malformed from the GitHub perspective.  They do not format as sub-paragraphs for line  items, and they ruin the formatting for later items. -->

```typescript

// These are required by the portal presently.  Re: Part Settings, the Part below works exclusively in terms of
// 'configuration.updateValues' to update settings values and 'onInputsSet(..., settings)' to receive settings values.
public colorSettingValue = ko.observable<BackgroundColor>();
public fontSettingValue = ko.observable<FontStyle>();

```

<!-- TODO:  Determine why the previous sample seems to be malformed from the GitHub perspective.  They do not format as sub-paragraphs for line  items, and they ruin the formatting for later items. -->

Retrieve the settings by using the blade container.

```typescript

const configuration = container.activateConfiguration<Settings>();
this.configureHotSpot = new HotSpotViewModel(container, {
    supplyBladeReference: () => {
        const bladeRef = new PdlTemplateBladeWithSettingsConfigurationReference<BladeConfiguration, BladeConfiguration>({
            // The Configuration values are sent to the Provider Blade to be edited by the user.
            supplyInitialData: () => {
                return configuration.getValues();
            },

            // The edited Configuration values are returned from the Provider Blade and updated in this Part.
            // Any edits will cause 'onInputsSet' to be called again, since this is the method where the Part receives a new, consistent
            // set of inputs/settings.
            receiveResult: (result) => {
                configuration.updateValues(result);
            }
        });

        bladeRef.metadata = {
            isContextBlade: true
        };

        return bladeRef;
    }
});

```

Also send the settings to the `onInputsSet` method.

```typescript

public onInputsSet(inputs: Def.TemplateBladeWithSettingsViewModel.InputsContract, settings: Def.TemplateBladeWithSettingsViewModel.SettingsContract): MsPortalFx.Base.Promise {
    // Any changes to the  Configuration values (see 'updateValues' above) will cause 'onInputsSet' to be called with the
    // new inputs/settings values.
    this._colorSetting(settings && settings.content && settings.content.colorSettingValue || BackgroundColor.Default);
    this._fontSetting(settings && settings.content && settings.content.fontSettingValue || FontStyle.Default);

    return null;
}

```

<a name="advanced-topics-displaying-unauthorized-ui"></a>
### Displaying Unauthorized UI

You can set the blade to Unauthorized UI using the `unauthorized` member of the blade container. The code that describes how to set the blade is located at  `<dir>/Client/V1/Blades/Unauthorized/ViewModels/UnauthorizedBladeViewModel.ts`.

<!-- TODO: Determine why it is a container and not a class. -->

The following code does this statically, but it can also be done dynamically, based  on a condition after data is loaded.

```typescript

constructor(container: MsPortalFx.ViewModels.ContainerContract,
            initialState: any,
            dataContext: BladesArea.DataContext) {
    super();
    this.title(ClientResources.bladeUnauthorized);
    this.subtitle(ClientResources.bladesLensTitle);

    //This call marks the Blade as unauthorized, which should display a specialized UI.
    // container.unauthorized();

    // Or display a specialized UI with a customized message
    container.unauthorized(ClientResources.bladeUnauthorizedCustomizedMessage);
}

```

<a name="advanced-topics-dynamically-displaying-notice-ui"></a>
### Dynamically displaying Notice UI

You can set the blade to the Notice UI using `enableNotice` member of the blade container. The code that describes how to set the blade is located at  `<dir>Client/V1/Blades/DynamicNotice/ViewModels/DynamicNoticeViewModels.ts`.

The blade can be enabled statically with the constructor, or it can be done dynamically. In the following example, the blade is set to the Notice UI if the **id** input parameter contains a specific value.

```typescript

public onInputsSet(inputs: any): MsPortalFx.Base.Promise {
    this.title(inputs.id);

    if (inputs.id === "42" || inputs.id === "43") {
        // to simulate the response from service and enable notice accordingly.
        return Q.delay(1000).then(() => {
            this._container.enableNotice({
                noticeTitle: ClientResources.comingSoonTitle,
                noticeHeader: ClientResources.comingSoon.format(inputs.id),
                noticeDescription: ClientResources.comingSoonDescription,
                noticeCallToActionText: ClientResources.comingSoonAction,
                noticeCallToActionUri: ClientResources.microsoftUri,
                noticeImageType: MsPortalFx.ViewModels.Controls.Notice.ImageType.ComingSoon
            });
        });
    } else {
        return null;
    }
}

```