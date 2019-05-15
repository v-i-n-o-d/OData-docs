---
title: "Breaking changes about validation settings"
description: ""

---

We used to have lots of validation related members/flags in `ODataMessageReaderSettings` and `ODataMessageWriterSettings`. In OData 7.0, we cleaned up the out-dated flags and put the remained flags together and keep them be considered in a consistent way.

## Removed APIs ##

| ODataMessageWriterSettings            | ODataMessageReaderSettings            |
|---------------------------------------|---------------------------------------|
| EnableFullValidation                  | EnableFullValidation                  |
| EnableDefaultBehavior()               | EnableDefaultBehavior()               |
| EnableODataServerBehavior()           | EnableODataServerBehavior()           |
| EnableWcfDataServicesClientBehavior() | EnableWcfDataServicesClientBehavior() |
|                                       | UndeclaredPropertyBehaviorKinds       |
|                                       | DisablePrimitiveTypeConversion        |
|                                       | DisableStrictMetadataValidation       |

The EnablexxxBehavior() in writer and reader settings actually wrapped few flags.

| ODataWriterBehavior                         | ODataReaderBehavior         |
|---------------------------------------------|-----------------------------|
| AllowDuplicatePropertyNames                 | AllowDuplicatePropertyNames |
| AllowNullValuesForNonNullablePrimitiveTypes |                             |

Those flags are all removed, and an enum type would represent all the settings instead.

## New API ##

A flag enum type ValidationKinds to represent all validation kinds in reader and writer:
```C#
/// <summary>
/// Validation kinds used in ODataMessageReaderSettings and ODataMessageWriterSettings.
/// </summary>
[Flags]
public enum ValidationKinds
{
    /// <summary>
    /// No validations.
    /// </summary>
    None = 0,

    /// <summary>
    /// Disallow duplicate properties in ODataResource (i.e., properties with the same name).
    /// If no duplication can be guaranteed, this flag can be turned off for better performance.
    /// </summary>
    ThrowOnDuplicatePropertyNames = 1,

    /// <summary>
    /// Do not support for undeclared property for non open type.
    /// </summary>
    ThrowOnUndeclaredPropertyForNonOpenType = 2,

    /// <summary>
    /// Validates that the type in input must exactly match the model.
    /// If the input can be guaranteed to be valid, this flag can be turned off for better performance.
    /// </summary>
    ThrowIfTypeConflictsWithMetadata = 4,

    /// <summary>
    /// Enable all validations.
    /// </summary>
    All = ~0
}
```

Writer: Add member Validations which accepts all the combinations of ValidationKinds
```C#
/// <summary>
/// Configuration settings for OData message writers.
/// </summary>
public sealed class ODataMessageWriterSettings
{

/// <summary>
/// Gets or sets Validations in writer.
/// </summary>
public ValidationKinds Validations { get; set; }

}
```

Reader: Add member Validations which accepts all the combinations of ValidationKinds
```C#
/// <summary>
/// Configuration settings for OData message readers.
/// </summary>
public sealed class ODataMessageReaderSettings
{

/// <summary>
/// Gets or sets Validations in reader.
/// </summary>
public ValidationKinds Validations { get; set; }

}
```

## Sample Usage ##
`writerSettings.Validations = ValidationKinds.All`<br />
Equal to:
`writerSettings.EnableFullValidation = true`

`readerSettings.Validations |= ValidationKinds.ThrowIfTypeConflictsWithMetadata`<br />
Equal to:
`readerSettings.DisableStrictMetadataValidation = false`

Same for reader.