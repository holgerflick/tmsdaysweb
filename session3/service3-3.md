---
title: "Web Service: Server"
parent: "Session 3: Multi-tier Web"
layout: default
nav_order: 3
has_children: false
nav_exclude: true
---

# Web Service: Server

```pascal
type
  [ServiceContract]
  IYardSaleService = interface(IInvokable)
    ['{7E7BEBC9-2BE3-425E-A579-590C3BE884A0}']

    function LoginAdmin( Login, Password: String ): TLoginResponse;
    function LoginParticipant( SaleId: Integer; Email, Name, Zip : String ): TLoginResponse;

    procedure AddParticipant( NewParticipant: TNewParticipant );
    function GetYardSale( SaleId: Integer ): TYardSale;

    // --- Participant operations
    [Authorize]
    procedure UpdateParticipant( Participant: TUpdateParticipant );

    [Authorize]
    procedure DeleteParticipant;

    function ItemCategories( SortOrder:TItemCategorySortOrder ) : TItemCategories;

    // --- Admin operations
    // [Authorize]
    [HttpGet] function GetParticipantsReportPdf( SaleId: Integer ): TStream;

    [Authorize]
    function GetYardSales: TYardSales;

    [Authorize]
    function GetYardSaleLogo(
      SaleId: Integer;
      Width, Height: Integer ): TBytes;

    [Authorize]
    function GetParticipants( SaleId: Integer ): TDetailedParticipants;

    [Authorize]
    function GetParticipantCategories(
      ParticipantId: Integer ): TParticipantCategories;
  end;
  ```

  ```pascal
  procedure TYardSaleService.AddParticipant(NewParticipant: TNewParticipant);
var
  LManager: TParticipantManager;

begin
  LManager := TParticipantManager.Create;
  try
    LManager.AddParticipant( NewParticipant );
  finally
    LManager.Free;
  end;
end;
```

```pascal
procedure TParticipantManager.AddParticipant(
  ANewParticipant: TNewParticipant);
var
  LQuery: TFDQuery;

begin
  LQuery := TDbController.Shared.GetQuery;
  try
    TParticipantSqlManager.AddParticipantQuery( LQuery, ANewParticipant );
    TXDataOperationContext.Current.Response.StatusCode := 204;
  finally
    LQuery.ReturnToPool;
  end;
end;
```

```pascal
class procedure TParticipantSqlManager.AddParticipantQuery(AQuery: TFDQuery;
  ANewParticipant: TNewParticipant);
var
  LParticipantId: Integer;

begin
  // check for duplicate first
  AQuery.SQL.Text := 'SELECT COUNT(*) AS c FROM SalesParticipant WHERE ' +
    'Email = :Email AND SalesId = :SalesId';
  AQuery.ParamByName('Email').AsString := ANewParticipant.Email;
  AQuery.ParamByName('SalesId').AsInteger := ANewParticipant.SaleId;
  AQuery.Open;
  if AQuery.FieldByName('c').AsInteger > 0 then
  begin
    raise EXDataHttpException.Create(409, 'Duplicate. Participant already signed up.');
  end;
  AQuery.Close;

  // add participant
  AQuery.SQL.Text := 'INSERT INTO SalesParticipant ' +
    '(Email, SalesId, Name, Street, Zip, City, State ) VALUES ' +
    '(:Email, :SalesId, :Name, :Street, :Zip, :City, :State )'
    ;
  AQuery.ParamByName('Name').AsString := ANewParticipant.Participant.Name;
  AQuery.ParamByName('Street').AsString := ANewParticipant.Participant.Street;
  AQuery.ParamByName('Zip').AsString := ANewParticipant.Participant.Zip;
  AQuery.ParamByName('City').AsString := ANewParticipant.Participant.City;
  AQuery.ParamByName('State').AsString := ANewParticipant.Participant.State;
  AQuery.ExecSQL;

  // determine id of new participant
  LParticipantId := AQuery.Connection.GetLastAutoGenValue('');

  // add categories
  AQuery.SQL.Text := 'INSERT INTO ParticipantItemCategories ' +
    '(IdParticipant, IdCategory, Comment) VALUES ' +
    '(:IdParticipant, :IdCategory, :Comment)'
    ;
  AQuery.ParamByName('IdParticipant').AsInteger := LParticipantId;
  for var LCategory in ANewParticipant.Categories do
  begin
    AQuery.ParamByName('IdCategory').AsInteger := LCategory.Id;
    AQuery.ParamByName('Comment').AsString := LCategory.Comment;
    AQuery.ExecSQL;
  end;
end;
```

```pascal
function TYardSaleService.GetYardSale(SaleId: Integer): TYardSale;
var
  LManager: TParticipantManager;

begin
  LManager := TParticipantManager.Create;
  try
    Result := LManager.GetYardSale( SaleId );
  finally
    LManager.Free;
  end;
end;
```

```pascal
function TParticipantManager.GetYardSale(ASaleId: Integer): TYardSale;
var
  LQuery: TFDQuery;

begin
  Result := TYardSale.Create;
  TXDataOperationContext.Current.Handler.ManagedObjects.Add(Result);

  LQuery := TDbController.Shared.GetQuery;
  try
    TParticipantSqlManager.YardSale( LQuery, ASaleId );
    LQuery.Open;

    if LQuery.Eof then
    begin
      raise EXDataHttpException.Create(404, 'Yard Sale not found.');
    end;

    Result.Transfer( LQuery );

  finally
    LQuery.ReturnToPool;
  end;
end;
```

```pascal
class procedure TParticipantSqlManager.YardSale(AQuery: TFDQuery;
  ASaleId: Integer);
begin
  AQuery.SQL.Text := 'SELECT * FROM YardSales WHERE Id = :Id';
  AQuery.ParamByName('Id').AsInteger := ASaleId;
end;
```

```pascal
procedure TYardSale.Transfer(AQuery: TFDQuery);
var
  LThumb: TBytes;

begin
    self.Id := AQuery.FieldByName('Id').AsInteger;
    self.EventStart := AQuery.FieldByName('EventStart').AsDateTime;
    self.EventEnd := AQuery.FieldByName('EventEnd').AsDateTime;
    self.Title := AQuery.FieldByName('title').AsString;

    // do we have a logo?
    if AQuery.FieldByName('logo').IsNull = False then
    begin
      // thumbnail available?
      if AQuery.FieldByName('thumb').IsNull then
      begin
        // generate thumbnail
        LThumb := TBitmapTools.Resize( AQuery.FieldByName('logo').AsBytes );

        // update database
        AQuery.Edit;
        AQuery.FieldByName('thumb').AsBytes := LThumb;
        AQuery.Post;
      end
      else
      begin
        // use thumbnail
        LThumb := AQuery.FieldByName('thumb').AsBytes;
      end;

      // assign thumbnail
      self.Logo := LThumb;
    end;
end;
```


```pascal
function TYardSaleService.ItemCategories(
  SortOrder: TItemCategorySortOrder ): TItemCategories;
var
  LManager: TParticipantManager;

begin
  LManager := TParticipantManager.Create;
  try
    Result := LManager.ItemCategories(SortOrder);
  finally
    LManager.Free;
  end;
end;
```

```pascal
class procedure TParticipantSqlManager.ItemCategories(
  LQuery: TFDQuery;
  ASortOrder: TItemCategorySortOrder );
begin
  if ASortOrder = soName then
  begin
    LQuery.SQL.Text := 'SELECT Id, Name FROM ItemCategories ORDER BY Name';
  end;

  if ASortOrder = soUsage then
  begin
    LQuery.SQL.Text :=
    'SELECT Id, NAME, COUNT(IdCategory) AS cnt FROM ItemCategories' +
    '  LEFT JOIN ParticipantItemCategories ON Id = IdCategory' +
    '  GROUP BY Id ' +
    '  ORDER BY cnt DESC';
  end;
end;
```




