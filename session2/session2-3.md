---
title: "Directions"
parent: "Session 2: TMS FNC Maps"
layout: default
nav_order: 3
has_children: false
---

# Directions

In order to visit all participants by car, we will calculate an optimal route with all participants as waypoints. Starting and end location is the same as we want to return to the office after visiting all of them.

This truly sounds very complex, but it is actually the easiest part of the whole session. It involves  knowledge of the classes and data structures that FNC Maps provides, but no particular coding skill.

```pascal
procedure TMainViewController.OptimizeRoute(ASalesId: Integer; AHome: String;
    AMap: TTMSFNCGoogleMaps);
var
  LWaypoints: TStringlist;
begin
  FParticipants.Free;
  FParticipants := FModel.GetParticipants(ASalesId);

  LWayPoints := TStringlist.Create;
  try
    // add all participants as waypoints
    for var LParticipant in FParticipants do
    begin
      LWaypoints.Add( LParticipant.Address + ', USA' );
    end;

    AMap.AddDirections( AHome, AHome, False, True, clRed, 2, 0.5, dtmDriving,
       True, LWaypoints, True );

  finally
    LWaypoints.Free;
  end;
end;
```

After getting all the participants of a sale, we need to add all their addresses as waypoints. The format for waypoints is a list of strings, a `TStringlist` to be precise. We can then call `AddDirections` specifying the origin and destination with the same address and the list of waypoints. That's it. Truly amazing how powerful this framework is...

We are not done yet, as, just like the geocoding process, the directions process is asynchronous. When it completes, the event `OnRetrievedDirectionsData` is called.

```pascal
procedure TFrmMain.MapRetrievedDirectionsData(Sender: TObject; AEventData:
    TTMSFNCMapsEventData; ADirectionsData: TTMSFNCGoogleMapsDirectionsData);
begin
  FLastDirections := ADirectionsData;

  FViewController.DisplayRoutes( Routes, ADirectionsData );
end;
```
There, we store the direction data in a field variable an call `DisplayRoutes` to show the information on screen.

The key is to read the correct information from the object derived from the certainly short class name `TTMSFNCGoogleMapsDirectionsData`.

{: .note}
Google Maps usually returns multiple routes. We are always going to show the first one without even looking at the other alternatives.

```pascal
 var LRoute := ADirectionsData.Routes[0];

  if Length( LRoute.Legs ) = 0 then
  begin
    exit;
  end;

  LSteps := 0;

  for var LLeg in LRoute.Legs do
  begin
    for var LStep in LLeg.Steps do
    begin
      Inc(LSteps);
      var LItem := ARoutes.Items.Add;
      LItem.Caption := LSteps.ToString;
      LItem.SubItems.Add( LStep.Distance.ToString + ' m' );
      LItem.SubItems.Add( LStep.Duration.ToString + ' min' );
      LItem.SubItems.Add( StripHTML( LStep.Instructions ) );
    end;
  end;
```

Pick the first route `LRoute`. Legs are parts of a route. Each leg has steps. Iterating these data structures allows us to print turn-by-turn instructions.

Sadly, Google only returns HTML content which is not interpret by the visual control we use. Thus, we need to remove all HTML tags. Delphi comes with regular expression support these days. Using a regular expression, we can strip any HTML.

```pascal
function TMainViewController.StripHtml(AText: String): String;
var
  R: TRegEx;
begin
  R := TRegEx.Create( '<([^>]+)>', [roIgnoreCase, roMultiLine] );
  Result := R.Replace( AText, ' ' );
  
  // replace duplicate spaces with one
  Result := Result.Replace('  ', ' ', [rfReplaceAll]);
end;
```