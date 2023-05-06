---
title: "Preview"
layout: default
nav_order: 3
has_children: false
parent: "Session 1: TMS FlexCel"
---

# Preview

The preview window will allow viewing the report. In addition, the end user will be able to export the report to different formats.

![](../images/01/previewdesign.png)

## Linking the report

The report needs to be known to the preview form. Thus, we use a field variable `FManager` to store a reference to an instance of `TReportManager`. The report manager will be passed a database connection. Thus, its queries can be tied to that connection.

```pascal
constructor TFrmPreview.Create(AModel: TDbModel);
begin
  inherited Create( nil );

  FModel := AModel;
  FManager := TReportManager.Create( nil, FModel.Connection );

  UpdateToolbar;

  FImgExport := nil;
end;
```
`FImgExport` is needed for the preview. A preview is generated using the image export from FlexCel. It is tied to the form window as it is not needed anywhere else. After setting everything up, `UpdateToolbar` is called to initialize the button states of the user interface.

## Showing the form

We define a public method to show the form. This makes it easy to add a parameter `ASalesId` to identify the yard sale we want to show the report for. Thus, the main view controller can show the preview of the report by passing an `Id` and nothing else.

```pascal
procedure TFrmPreview.ShowParticipants(ASalesId: Integer);
begin
  FManager.ReportParticipants(ASalesId);
  UpdateReport;
  UpdateToolbar;

  self.ShowModal;
end;
```

Before showing the form the report is generated. 

{: .note}
In case the report generation takes a long time, you need to build a different construct. For example, create a method called `Prepare` and report progress back to the call site. Then, when completed, the caller can show the form.

After the report has been generated, we update the button states and update the preview. As we are inside of a VCL application all these calls are serialized.

```pascal
procedure TFrmPreview.UpdateToolbar;
var
  LEnabled: Boolean;

begin
  LEnabled := FManager.LastReport <> nil;

  btnHtml.Enabled := LEnabled;
  btnPdf.Enabled := LEnabled;
  btnXls.Enabled := LEnabled;
end;
```

The buttons that will allow us to export a report ought only be enabled if a report has been generated. `LastReport` of the report manager gives us that information. If it is not `nil`, a report has been generated and the buttons can be enabled.

## Generating the preview

```pascal
procedure TFrmPreview.UpdateReport;
begin
  if Assigned( FManager.LastReport ) then
  begin
    FImgExport.Free;

    // create an image export as import for preview
    FImgExport := TFlexCelImgExport.Create( FManager.LastReport );

    // assign the image export
    Preview.Document := FImgExport;

    // update the window
    Preview.InvalidatePreview;
  end;
end;
```

## Exporting reports

All exports work the same way:

- Initialize the Save As-dialog
- Prompt the user to enter a filename
- Instantiate an export class of the corresponding file type with the last generated report
- Call its `Export` method with the filename the user entered

Done! 😃

### HTML

```pascal
procedure TFrmPreview.ExportHtml;
var
  LFileType: TFileTypeItem;
  LExport: TFlexCelHtmlExport;

begin
  DlgSave.FileTypes.Clear;
  DlgSave.DefaultExtension := 'html';
  LFileType := DlgSave.FileTypes.Add;

  LFileType.DisplayName := 'Hypertext Markup Language Format (*.html)';
  LFileType.FileMask := '*.html';

  if DlgSave.Execute then
  begin
    LExport := TFlexCelHtmlExport.Create(FManager.LastReport);
    try
      LExport.Export(DlgSave.FileName, 'images' );
    finally
      LExport.Free;
    end;
  end;
end;
```

### Microsoft Excel (*.xlsx)

```pascal
procedure TFrmPreview.ExportXls;
var
  LFileType: TFileTypeItem;
  LStream: TFileStream;

begin
  if Assigned( FManager.LastReport ) then
  begin
    DlgSave.FileTypes.Clear;
    DlgSave.DefaultExtension := 'xlsx';
    LFileType := DlgSave.FileTypes.Add;

    LFileType.DisplayName := 'Microsoft Excel (*.xlsx)';
    LFileType.FileMask := '*.xlsx';

    if DlgSave.Execute then
    begin
      FManager.LastReport.Save(DlgSave.FileName);
    end;
  end;
end;
```


### PDF

```pascal
procedure TFrmPreview.ExportPdf;
var
  LFileType: TFileTypeItem;
  LExport: TFlexCelPdfExport;

begin
  DlgSave.FileTypes.Clear;
  DlgSave.DefaultExtension := 'pdf';
  LFileType := DlgSave.FileTypes.Add;

  LFileType.DisplayName := 'Adobe Portable Document Format  (*.pdf)';
  LFileType.FileMask := '*.pdf';

  LExport := nil;

  if DlgSave.Execute then
  begin
    LExport := TFlexCelPdfExport.Create( FManager.LastReport );
    try
      LExport.Export( DlgSave.FileName );
    finally
      LExport.Free;
    end;
  end;
end;
```