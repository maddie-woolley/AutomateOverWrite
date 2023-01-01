import arcpy
import os
import xml.dom.minidom as DOM


def ScriptTool(map, service, summary, tags, description, overwriteService, enableEditing, enableSync, enableWFS,
               timezone, share_public, share_organization, share_groups, outdir):
    """ScriptTool function docstring"""

    # Set output file names
    sddraft_filename = service + ".sddraft"
    sddraft_output_filename = os.path.join(outdir, sddraft_filename)

    # Reference map to publish
    aprx = arcpy.mp.ArcGISProject("CURRENT")
    m = aprx.listMaps(map)[0]

    # Create FeatureSharingDraft and set service properties
    sharing_draft = m.getWebLayerSharingDraft("HOSTING_SERVER", "FEATURE", service)
    sharing_draft.summary = summary
    sharing_draft.tags = tags
    sharing_draft.description = description
    sharing_draft.credits = "My Credits"
    sharing_draft.useLimitations = "My Use Limitations"
    sharing_draft.overwriteExistingService = overwriteService

    # Create Service Definition Draft file
    sharing_draft.exportToSDDraft(sddraft_output_filename)
    outsddraft = sddraft_output_filename
    arcpy.AddMessage("Service definition draft created")

    # Modify capabilities
    if enableEditing or enableSync:
        ModifyCapabilities(sddraft_output_filename, enableEditing, enableSync)
    if enableWFS:
        EnableWFS(sddraft_output_filename)

    # Set time zone
    if (timezone != ""):
        property_set = [{
            "key": "dateFieldsRespectsDayLightSavingTime",
            "value": "true"
        },
            {
                "key": "dateFieldsTimezoneID",
                "value": timezone
            }]
        SetTimezone(sddraft_output_filename, property_set=property_set)

    # Create Service Definition file
    sd_filename = service + ".sd"
    sd_output_filename = os.path.join(outdir, sd_filename)
    arcpy.StageService_server(sddraft_output_filename, sd_output_filename)
    arcpy.AddMessage("Service definition created")

    # Upload to portal
    output = arcpy.UploadServiceDefinition_server(sd_output_filename, "My Hosted Services",
                                                  in_override="OVERRIDE_DEFINITION", in_public=share_public,
                                                  in_organization=share_organization, in_groups=share_groups)
    arcpy.AddMessage("Service published")

    return output[5]


def ModifyCapabilities(sddraft_output_filename, enableEditing, enableSync):
    capabilities = "Query"
    if enableEditing:
        capabilities += ",Create,Delete,Update,Editing"
    if enableSync:
        capabilities += ",Sync"
    # Modify feature layer capabilities to enable Create and Sync
    doc = DOM.parse(sddraft_output_filename)
    typeNames = doc.getElementsByTagName('TypeName')
    for typeName in typeNames:
        if typeName.firstChild.data == "FeatureServer":
            extension = typeName.parentNode
            for extElement in extension.childNodes:
                if extElement.tagName == 'Definition':
                    for propArray in extElement.childNodes:
                        if propArray.tagName == 'Info':
                            for propSet in propArray.childNodes:
                                for prop in propSet.childNodes:
                                    for prop1 in prop.childNodes:
                                        if prop1.tagName == "Key":
                                            if prop1.firstChild.data == 'webCapabilities':
                                                if prop1.nextSibling.hasChildNodes():
                                                    prop1.nextSibling.firstChild.data = capabilities
                                                else:
                                                    txt = doc.createTextNode(capabilities)
                                                    prop1.nextSibling.appendChild(txt)
    # Write to the .sddraft file
    f = open(sddraft_output_filename, 'w')
    doc.writexml(f)
    f.close()
    arcpy.AddMessage("Capabilities updated")
    return


def EnableWFS(sddraft_output_filename):
    doc = DOM.parse(sddraft_output_filename)
    typeNames = doc.getElementsByTagName('TypeName')
    for typeName in typeNames:
        # Get the TypeName to enable
        if typeName.firstChild.data == "EnableWFSServer":
            extension = typeName.parentNode
            for extElement in extension.childNodes:
                # Enable feature access
                if extElement.tagName == 'Enabled':
                    extElement.firstChild.data = 'true'
    # Write to the .sddraft file
    f = open(sddraft_output_filename, 'w')
    doc.writexml(f)
    f.close()
    arcpy.AddMessage("WFS set")
    return


def SetTimezone(sddraftPath, property_set):
    # Read the sddraft xml
    doc = DOM.parse(sddraftPath)

    # Find all elements named TypeName. This is where the server object extension
    # (SOE) names are defined.
    typeNames = doc.getElementsByTagName('TypeName')
    for typeName in typeNames:
        # Get the TypeName to enable
        if typeName.firstChild.data == "MapServer":
            extension = typeName.parentNode
            # prp = extension.childNodes.getElementsByTagNameNS('PropertyArray')
            for extElement in extension.childNodes:
                if extElement.tagName == 'Definition':
                    for definition in extElement.childNodes:
                        if definition.tagName == 'ConfigurationProperties':
                            for config_prop in definition.childNodes:
                                if config_prop.tagName == 'PropertyArray':
                                    for prop in property_set:
                                        prop_set = doc.createElement("PropertySetProperty")
                                        attr = doc.createAttribute("xsi:type")
                                        attr.value = "typens:PropertySetProperty"
                                        prop_set.setAttributeNode(attr)

                                        prop_key = doc.createElement("Key")
                                        txt = doc.createTextNode(prop["key"])
                                        prop_key.appendChild(txt)
                                        prop_set.appendChild(prop_key)

                                        prop_value = doc.createElement("Value")
                                        attr = doc.createAttribute("xsi:type")
                                        attr.value = "xs:string"
                                        prop_value.setAttributeNode(attr)
                                        txt = doc.createTextNode(prop["value"])
                                        prop_value.appendChild(txt)
                                        prop_set.appendChild(prop_value)

                                        config_prop.appendChild(prop_set)

    # Write to the .sddraft file
    f = open(sddraftPath, 'w')
    doc.writexml(f)
    f.close()
    arcpy.AddMessage("Timezone set")
    return


if __name__ == '__main__':
    # ScriptTool parameters
    map = arcpy.GetParameter(0)
    service = arcpy.GetParameterAsText(1)
    summary = arcpy.GetParameterAsText(2)
    tags = arcpy.GetParameterAsText(3)
    description = arcpy.GetParameterAsText(4)
    overwriteService = arcpy.GetParameter(5)
    enableEditing = arcpy.GetParameter(6)
    enableSync = arcpy.GetParameter(7)
    enableWFS = arcpy.GetParameter(8)
    timezone = arcpy.GetParameterAsText(9)
    share_public = arcpy.GetParameterAsText(10)
    share_organization = arcpy.GetParameterAsText(11)
    share_groups = arcpy.GetParameterAsText(12)
    outdir = arcpy.GetParameterAsText(13)

    rest_endpoint = ScriptTool(map, service, summary, tags, description, overwriteService, enableEditing, enableSync,
                               enableWFS, timezone, share_public, share_organization, share_groups, outdir)
    arcpy.SetParameterAsText(1, rest_endpoint)
    arcpy.AddMessage(rest_endpoint)
