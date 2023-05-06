---
title: "Markers"
parent: "Session 2: TMS FNC Maps"
layout: default
nav_order: 2
has_children: false
---

# Custom Markers

Participants need to be easily located on the map. The map should also focus on the area of all participants. You will notice a steep decline in the complexity compared to the last part of the session. Geocoding truly is the most complex process due to its asynchronicity.

As we have all the coordinates of all the participants, we need to create a list of all participants with their locations and add one marker for each element of that list.

Each marker will be customized using the following icon instead of the default icon that Google provides.

> ![](../images/02/sale.png)

The image will be linked into the executable as a resource. A process which has been used for the Google API as well as for the FlexCel template before.

```
SALEPNG RCDATA ".\resources\sale_32.png"
```

The image has the dimensions 32x32 pixels.

## Preparing the custom icon

{: .note}
I will refer to the PNG image we use as an icon even though formally it is not stored in ICO format or similar.  

The icon can should not be loaded from a file. The Google Maps API cannot access our local file system without jumping through hoops. I am aware that TMS has provided a solution to access files on the local file system to work with Google Maps, but stay with me here as this solution will be bullet proof for any platform.

The solution we will implement is using a data URL. Data URLs are strings representing the contents the URL links to. Think of them as resources of the Web. If you do not want to get too deep into this topic, just use the method that I provide to convert a PNG image into a data URL.

You need to know, however, that a data URL encodes binary content as a Base64 string. With Delphi 11 this is very simple to achieve using its `TBase64Encoding` class.

```pascal
function TMainViewController.GetDataUrlForSaleIcon: String;
var
  LResource: TResourceStream;
  LOutput: TStringStream;

begin
  LResource := TResourceStream.Create( hInstance, 'SALEPNG', RT_RCDATA);
  LOutput := nil;
  try
    LOutput := TStringStream.Create;
    TBase64Encoding.Base64.Encode( LResource, LOutput );

    Result := 'data:image/png;base64,' + LOutput.DataString;
  finally
    LOutput.Free;
    LResource.Free;
  end;
end;
```

First, we extract the icon from the resource and pass that exact stream to `TBase64Encoding.Base64.Encode`. In order to retrieve a `String`, an output stream `LOutput` of type `TStringStream` is used. The data URL then needs to specify a MIME type of its encoded data which is `image/png` in this case.

For a detailed explanation of data URL, please have a look at the second edition of my TMS WEB Core book. 

`GetDataUrlForSaleIcon` returns our image as a string and that is exactly what we can pass as an URL to the Google Maps API! If at any point you need a URL and you want to specify an image, you can use a data URL like we just created instead.

## Adding markers 

```pascal
procedure TMainViewController.AddParticipants(ASalesId: Integer; AMap:
    TTMSFNCGoogleMaps);
var
  LIconDataUrl: String;

begin
  // get data url
  LIconDataUrl := GetDataUrlForSaleIcon;

  // get all participants for the sale
  FParticipants.Free;
  FParticipants := FModel.GetParticipants(ASalesId);

  AMap.BeginUpdate;
  try
    // clear map
    AMap.Clear;

    // iterate all participants
    for var LParticipant in FParticipants do
    begin
      // only add marker if location has been added
      if Assigned( LParticipant.Location ) then
      begin
        var LMarker := AMap.AddMarker(
          LParticipant.Location.Latitude,
          LParticipant.Location.Longitude,
          LParticipant.Name
          );

        // assign the object instance
        LMarker.DataObject := LParticipant;

        // assign custom icon
        LMarker.IconURL := LIconDataUrl;
      end;
    end;

    // if at least one marker exists
    if AMap.Markers.Count > 0 then
    begin
      // zoom in and change view to area of interest
      AMap.ZoomToBounds( AMap.Markers.ToCoordinateArray );
    end;
  finally
    AMap.EndUpdate;
  end;
end;
```
Adding markers to a map is pretty straightforward. Call `AddMarker` with the coordinates and a title for the marker to create a new marker instance. In this case, we add the participant object instance as `DataObject` to the marker so we are even able to determine which participant has been clicked when a marker is clicked. Further, we assign `IconURL`. Instead of a URL, we assign the data URL of the image which works just as well. The image just does not have to be on a server on the Internet. Instead, it was bundled with our application as a resource.

{: .note}
It is always good practice to call `BeginUpdate` and `EndUpdate` when adding a lot of markers to the map or making a lot of changes in general.

Finally, after adding all the markers, we want to focus the user's attention to the area of interest where all participants are located.

```pascal
AMap.ZoomToBounds( AMap.Markers.ToCoordinateArray );
```

`ZoomToBounds` finds an optimal zoom factor that all markers are visible and also pans the view that they will be visible. `Markers` is a list of all markers that offers a property `ToCoordinateArray` to convert all coordinates into an array which `ZoomToBounds` expects as input.