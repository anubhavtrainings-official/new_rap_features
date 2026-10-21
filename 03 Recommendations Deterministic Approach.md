```
@AbapCatalog.viewEnhancementCategory: [#NONE]
@AccessControl.authorizationCheck: #NOT_REQUIRED
@EndUserText.label: 'Item Text Value Help'
@Metadata.ignorePropagatedAnnotations: true
define view entity ZI_ATS_AB_ITEMTEXT_VH as select from zats_ab_equip
{
    key item_text as ItemText,
    key category_id as CategoryId
} where item_text <> ''

```
----
```
@AccessControl.authorizationCheck: #MANDATORY
@Metadata.allowExtensions: true
@ObjectModel.sapObjectNodeType.name: 'ZATS_AB_EQUIP'
@EndUserText.label: 'Equipment Request'
define root view entity ZR_ATS_AB_EQUIP
  as select from zats_ab_equip
  association [0..1] to ZATS_AB_PRODCAT as _Category on $projection.CategoryId    = _Category.Value
  association [0..1] to ZATS_AB_COMPANY as _Supplier on $projection.SupplierName  = _Supplier.Value
  association [0..1] to ZATS_AB_STATUS  as _Status   on $projection.OverallStatus = _Status.Value
{
      @EndUserText.label: 'Order UUID'
  key order_uuid            as OrderUUID,
      @EndUserText.label: 'Item Name'
      @Consumption.valueHelpDefinition: [{ entity: { name: 'ZI_ATS_AB_ITEMTEXT_VH', element: 'ItemText' },
                                           additionalBinding: [{ 
                                                localElement: 'CategoryId',
                                                element: 'CategoryId',
                                                usage: #FILTER
                                           }],
                                         useForValidation: false }]
      item_text             as ItemText,
      @EndUserText.label: 'Product Category'
      @ObjectModel.text.element: [ 'CategoryText' ]
      @Consumption.valueHelpDefinition: [{ entity: { name: 'ZATS_AB_PRODCAT', element: 'Value' },
                                         useForValidation: true }]
      category_id           as CategoryId,
      @EndUserText.label: 'Category Text'
      _Category.Description as CategoryText,

      @EndUserText.label: 'Supplier Name'
      @ObjectModel.text.element: [ 'SupplierText' ]
      @Consumption.valueHelpDefinition: [{ entity: { name: 'ZATS_AB_COMPANY', element: 'Value' },
                                           useForValidation: true }]
      supplier_name         as SupplierName,
      @EndUserText.label: 'Supplier Text'
      _Supplier.Description as SupplierText,

      @EndUserText.label: 'Unit Price'
      @Semantics.amount.currencyCode: 'CurrencyCode'
      unit_price            as UnitPrice,
      @EndUserText.label: 'Currency'
      @Consumption.valueHelpDefinition: [ {
        entity.name: 'I_CurrencyStdVH',
        entity.element: 'Currency',
        useForValidation: true
      } ]


      currency_code         as CurrencyCode,
      @EndUserText.label: 'Quantity'
      quantity              as Quantity,
      @EndUserText.label: 'Delivery Date'
      delivery_date         as DeliveryDate,
      @EndUserText.label: 'Overall Status'
      @ObjectModel.text.element: [ 'StatusText' ]
      @Consumption.valueHelpDefinition: [{ entity: { name: 'ZATS_AB_STATUS', element: 'Value' },
                                         useForValidation: true }]
      overall_status        as OverallStatus,
      @EndUserText.label: 'Status Text'
      _Status.Description   as StatusText,
      @EndUserText.label: 'Review'
      review                as Review,
      @EndUserText.label: 'Created By'
      @Semantics.user.createdBy: true
      local_created_by      as LocalCreatedBy,
      @EndUserText.label: 'Created At'
      @Semantics.systemDateTime.createdAt: true
      local_created_at      as LocalCreatedAt,
      @EndUserText.label: 'Last Changed By'
      @Semantics.user.localInstanceLastChangedBy: true
      local_last_changed_by as LocalLastChangedBy,
      @EndUserText.label: 'Local Last Changed At'
      @Semantics.systemDateTime.localInstanceLastChangedAt: true
      local_last_changed_at as LocalLastChangedAt,
      @EndUserText.label: 'Last Changed At'
      @Semantics.systemDateTime.lastChangedAt: true
      last_changed_at       as LastChangedAt,

      /* Associations */
      _Category,
      _Supplier,
      _Status
}

```


----
```
managed implementation in class ZBP_R_ATS_AB_EQUIP unique;
strict ( 2 );
with draft;
extensible;
define behavior for ZR_ATS_AB_EQUIP alias ZrAtsAbEquip
persistent table ZATS_AB_EQUIP
extensible
draft table ZATS_AB_EQUIP_D
etag master LocalLastChangedAt
lock master total etag LastChangedAt
authorization master( global )
with additional save with full data
{
  field ( readonly )
   OrderUUID,
   LocalCreatedBy,
   LocalCreatedAt,
   LocalLastChangedBy,
   LocalLastChangedAt,
   LastChangedAt;

  field ( numbering : managed )
   OrderUUID;

  action (features : instance) approveRequest result [1] $self;

  create;
  update;
  delete;

  event eqip_req_accepted parameter ZA_ATS_AB_EQUIPMENT;
  event StatusUpdated for side effects;

  //add logic for supporting recommendations
  field (recommendations) ItemText;
  recommendation function getItemTexts;

  side effects {
    event StatusUpdated affects field ( OverallStatus, Review );
    recommendation function getItemTexts
        executed on field CategoryId;
  }



  draft action Activate optimized;
  draft action Discard;
  draft action Edit;
  draft action Resume;
  draft determine action Prepare;

  mapping for ZATS_AB_EQUIP corresponding extensible
  {
    OrderUUID = ORDER_UUID;
    ItemText = ITEM_TEXT;
    CategoryID = CATEGORY_ID;
    SupplierName = SUPPLIER_NAME;
    UnitPrice = UNIT_PRICE;
    CurrencyCode = CURRENCY_CODE;
    Quantity = QUANTITY;
    DeliveryDate = DELIVERY_DATE;
    OverallStatus = OVERALL_STATUS;
    Review = REVIEW;
    LocalCreatedBy = LOCAL_CREATED_BY;
    LocalCreatedAt = LOCAL_CREATED_AT;
    LocalLastChangedBy = LOCAL_LAST_CHANGED_BY;
    LocalLastChangedAt = LOCAL_LAST_CHANGED_AT;
    LastChangedAt = LAST_CHANGED_AT;
  }

}
```


-----

```

projection implementation in class ZBP_C_ATS_AB_EQUIP unique;
strict ( 2 );
extensible;
use draft;
use side effects;
use recommendations;
define behavior for ZC_ATS_AB_EQUIP alias ZcAtsAbEquip
extensible
use etag
{
  use create;
  use update;
  use delete;

  use action approveRequest;
  use event StatusUpdated;

  use action Edit;
  use action Activate;
  use action Discard;
  use action Resume;
  use action Prepare;

}
```
----
```
CLASS lsc_zr_ats_ab_equip DEFINITION INHERITING FROM cl_abap_behavior_saver.

  PROTECTED SECTION.

    METHODS save_modified REDEFINITION.

ENDCLASS.

CLASS lsc_zr_ats_ab_equip IMPLEMENTATION.

  METHOD save_modified.

    DATA : events_to_be_raied type table for event ZR_ATS_AB_EQUIP~eqip_req_accepted.


    "Raise the event whenever the status is changed
    if update-zratsabequip is not initial.

        LOOP at update-zratsabequip into data(update_equipment).

            clear events_to_be_raied.

            if update_equipment-%control-OverallStatus = if_abap_behv=>mk-on and
                update_equipment-OverallStatus = 'I'.

                data(process_monitor_string) = zcl_ats_ab_orch=>run_via_bgpf_tx_uncontrolled(

                        i_rap_bo_entity =  update_equipment

                        ).

            ENDIF.

            if update_equipment-%control-OverallStatus = if_abap_behv=>mk-on.

                data : status_updated type table for event ZR_ATS_AB_EQUIP~StatusUpdated.

                APPEND INITIAL LINE TO status_updated.
                status_updated[ 1 ] = CORRESPONDING #( update_equipment ).
                raise entity event ZR_ATS_AB_EQUIP~StatusUpdated from status_updated.

            endif.

            if update_equipment-%control-OverallStatus = if_abap_behv=>mk-on and
                update_equipment-OverallStatus = 'A'.

                append INITIAL LINE TO events_to_be_raied.
                events_to_be_raied[ 1 ] = CORRESPONDING #( update_equipment
                                                                mapping
                                                                %key = %key
                                                                order_uuid = OrderUUID ).
                raise entity event ZR_ATS_AB_EQUIP~eqip_req_accepted from events_to_be_raied.

            ENDIF.

        endloop.

    endif.


  ENDMETHOD.

ENDCLASS.

CLASS LHC_ZR_ATS_AB_EQUIP DEFINITION INHERITING FROM CL_ABAP_BEHAVIOR_HANDLER.
  PRIVATE SECTION.
    METHODS:
      GET_GLOBAL_AUTHORIZATIONS FOR GLOBAL AUTHORIZATION
        IMPORTING
           REQUEST requested_authorizations FOR ZrAtsAbEquip
        RESULT result,
      get_instance_features FOR INSTANCE FEATURES
            keys REQUEST requested_features FOR ZrAtsAbEquip RESULT result.

          METHODS approveRequest FOR MODIFY
            keys FOR ACTION ZrAtsAbEquip~approveRequest RESULT result.
          METHODS getItemTexts FOR READ
            keys FOR FUNCTION ZrAtsAbEquip~getItemTexts RESULT result.
ENDCLASS.

CLASS LHC_ZR_ATS_AB_EQUIP IMPLEMENTATION.
  METHOD GET_GLOBAL_AUTHORIZATIONS.
  ENDMETHOD.
  METHOD get_instance_features.

  "Step 1: Read the travel data with status
   READ ENTITIES OF ZR_ATS_AB_EQUIP in local mode
       ENTITY ZrAtsAbEquip
           FIELDS ( OrderUUID overallstatus  )
           with     CORRESPONDING #( keys )
       RESULT data(requests)
       FAILED failed.

   result = value #( for requestw in requests
                       ( %tky = requestw-%tky
                         %action-approveRequest = COND #( WHEN requestw-OverallStatus = 'A'
                                                           then if_abap_behv=>fc-o-disabled
                                                           else if_abap_behv=>fc-o-enabled
                         )

                       )
                   ).

  ENDMETHOD.

  METHOD approveRequest.

     ""Perform the change of BO instance to change status
   MODIFY ENTITIES OF ZR_ATS_AB_EQUIP
       ENTITY ZrAtsAbEquip
           UPDATE FIELDS ( OverallStatus )
           WITH VALUE #( for key in keys ( %tky = key-%tky
                                           %is_draft = key-%is_draft
                                           OverallStatus = 'I'
            )  ).
   ""Read the BO instance on which we want to make the changes
   READ ENTITIES OF ZR_ATS_AB_EQUIP
       ENTITY ZrAtsAbEquip
           ALL FIELDS
               WITH CORRESPONDING #( keys )
                   RESULT data(lt_results).
   result = value #( for travel in lt_results ( %tky = travel-%tky
                                                %param = travel
   ) ).

  ENDMETHOD.


  METHOD getItemTexts.

    data : lv_orderuid type zats_ab_equip-order_uuid,
           lv_ordercat type zats_ab_equip-category_id.

    READ ENTITIES OF ZR_ATS_AB_EQUIP IN LOCAL MODE
        entity ZrAtsAbEquip
        fields ( OrderUUID CategoryId )
        WITH CORRESPONDING #( keys )
        RESULT data(lt_read_result).


    ASSIGN lt_read_result[ 1 ] to field-SYMBOL(<s_read_result>).

    if sy-subrc = 0.
        lv_orderuid = <s_read_result>-OrderUUID.
        lv_ordercat = <s_read_result>-CategoryId.
    endif.

    if lv_ordercat is not initial.
        data(lt_items) = zcl_ats_ab_util_tabai=>get_item_suggestions(
          EXPORTING
            iv_category_id = lv_ordercat
        ).
    endif.

    " %tky already contains %pky, so we assign the %tky itself
    result = value #( (
                        %tky = keys[ 1 ]-%tky
                        %param-zratsabequip  = value #(
                            itemtext = cond #( when lines( lt_items ) > 0 and
                                                    lv_ordercat is not initial
                                                    then CORRESPONDING #( lt_items )
                                                    else value #(  )
                                             )
                        )

    ) ).




  ENDMETHOD.

ENDCLASS.


```

----
```
CLASS zcl_ats_ab_util_tabai DEFINITION
  PUBLIC
  FINAL
  CREATE PUBLIC .

  PUBLIC SECTION.

    TYPES: BEGIN OF ty_item_text_suggestion,
            value type string,
            score TYPE decfloat34,
            suggested TYPE abap_bool,
           end of ty_item_text_suggestion,
           tt_item_suggestion type standard table of ty_item_text_suggestion
                with empty key.

    constants: c_top_rows type i value 2,
               c_score_high type decfloat34 value '0.8',
               c_score_medium type decfloat34 value '0.6',
               c_score_low type decfloat34 value '0.2'.

    CLASS-methods get_item_suggestions
        IMPORTING iv_category_id TYPE zats_ab_equip-category_id
        RETURNING VALUE(rt_suggestions) type tt_item_suggestion.

  PROTECTED SECTION.
  PRIVATE SECTION.
ENDCLASS.



CLASS zcl_ats_ab_util_tabai IMPLEMENTATION.
  METHOD get_item_suggestions.

    select from zats_ab_equip
        fields item_text as value,
        case
            when sum( quantity ) > 10 then @c_score_high
            when sum( quantity ) >= 5 then @c_score_medium
            else @c_score_low
            end as score,
            @abap_true as suggested,
            sum( quantity ) as total_qty
        where category_id = @iv_category_id
              and item_text <> @space
              and quantity > 0
        GROUP by item_text
        ORDER BY score descending, total_qty descending, value ascending
        into CORRESPONDING FIELDS OF table @rt_suggestions
        UP to @c_top_rows ROWS.



  ENDMETHOD.

ENDCLASS.



```