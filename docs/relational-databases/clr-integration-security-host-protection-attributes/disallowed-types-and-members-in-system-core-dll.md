---
title: "Disallowed Types and Members in System.Core.dll"
description: SQL Server CLR programming disallows a type or member with certain values for the HostProtectionResource enum. This article lists System.Core.dll disallowed values.
author: rwestMSFT
ms.author: randolphwest
ms.date: 12/27/2024
ms.service: sql
ms.subservice: clr
ms.topic: "reference"
---
# Disallowed types and members in System.Core.dll

[!INCLUDE [SQL Server](../../includes/applies-to-version/sqlserver.md)]

[!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] common language integration (CLR) programming disallows the use of a type or member that has a `HostProtectionAttribute` that specifies a `System.Security.Permissions.HostProtectionResource` enumeration with a value of `ExternalProcessMgmt`, `ExternalThreading`, `MayLeakOnAbort`, `SecurityInfrastructure`, `SelfAffectingProcessMgmnt`, `SelfAffectingThreading`, `SharedState`, `Synchronization`, or `UI`. The following table lists the members and types of the System.Core.dll assemblies whose Host Protection Attribute (HPA) values are disallowed.

> [!NOTE]  
> This list was generated from the supported assemblies. For more information, see [Supported .NET Framework libraries](../clr-integration/database-objects/supported-net-framework-libraries.md).

| Type or member | HPA values |
| --- | --- |
| `System.Diagnostics.Eventing.EventDescriptor` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.EventProvider` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.EventProviderTraceListener` | `MayLeakOnAbort` |
| `System.Management.Instrumentation.ManagementEntityAttribute` | `MayLeakOnAbort` |
| `System.Management.Instrumentation.WmiConfigurationAttribute` | `MayLeakOnAbort` |
| `System.Management.Instrumentation.ManagementMemberAttribute` | `MayLeakOnAbort` |
| `System.Management.Instrumentation.ManagementNewInstanceAttribute` | `MayLeakOnAbort` |
| `System.Management.Instrumentation.ManagementBindAttribute` | `MayLeakOnAbort` |
| `System.Management.Instrumentation.ManagementCreateAttribute` | `MayLeakOnAbort` |
| `System.Management.Instrumentation.ManagementRemoveAttribute` | `MayLeakOnAbort` |
| `System.Management.Instrumentation.ManagementEnumeratorAttribute` | `MayLeakOnAbort` |
| `System.Management.Instrumentation.ManagementProbeAttribute` | `MayLeakOnAbort` |
| `System.Management.Instrumentation.ManagementTaskAttribute` | `MayLeakOnAbort` |
| `System.Management.Instrumentation.ManagementKeyAttribute` | `MayLeakOnAbort` |
| `System.Management.Instrumentation.ManagementReferenceAttribute` | `MayLeakOnAbort` |
| `System.Management.Instrumentation.ManagementConfigurationAttribute` | `MayLeakOnAbort` |
| `System.Management.Instrumentation.ManagementCommitAttribute` | `MayLeakOnAbort` |
| `System.Management.Instrumentation.ManagementNameAttribute` | `MayLeakOnAbort` |
| `System.Management.Instrumentation.InstrumentationBaseException` | `MayLeakOnAbort` |
| `System.Management.Instrumentation.InstrumentationException` | `MayLeakOnAbort` |
| `System.Management.Instrumentation.InstanceNotFoundException` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventBookmark` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventLogConfiguration` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventLogLink` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventLogStatus` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventProperty` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventLogPropertySelector` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventRecord` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventKeyword` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventLevel` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventLogRecord` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventLogReader` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventLogWatcher` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventRecordWrittenEventArgs` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventLogSession` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventMetadata` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventOpcode` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventTask` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventLogException` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventLogNotFoundException` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventLogReadingException` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventLogProviderDisabledException` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventLogInvalidDataException` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.EventLogInformation` | `MayLeakOnAbort` |
| `System.Diagnostics.Eventing.Reader.ProviderMetadata` | `MayLeakOnAbort` |
| `Microsoft.Win32.SafeHandles.SafeNCryptHandle` | `MayLeakOnAbort` |
| `Microsoft.Win32.SafeHandles.SafeNCryptKeyHandle` | `MayLeakOnAbort` |
| `Microsoft.Win32.SafeHandles.SafeNCryptProviderHandle` | `MayLeakOnAbort` |
| `Microsoft.Win32.SafeHandles.SafeNCryptSecretHandle` | `MayLeakOnAbort` |
| `System.Security.Cryptography.Aes` | `MayLeakOnAbort` |
| `System.Security.Cryptography.AesCryptoServiceProvider` | `MayLeakOnAbort` |
| `System.Security.Cryptography.AesManaged` | `MayLeakOnAbort` |
| `System.Security.Cryptography.CngAlgorithm` | `MayLeakOnAbort` |
| `System.Security.Cryptography.CngAlgorithmGroup` | `MayLeakOnAbort` |
| `System.Security.Cryptography.CngKey` | `MayLeakOnAbort` |
| `System.Security.Cryptography.CngKeyBlobFormat` | `MayLeakOnAbort` |
| `System.Security.Cryptography.CngKeyCreationParameters` | `MayLeakOnAbort` |
| `System.Security.Cryptography.CngProperty` | `MayLeakOnAbort` |
| `System.Security.Cryptography.CngPropertyCollection` | `MayLeakOnAbort` |
| `System.Security.Cryptography.CngProvider` | `MayLeakOnAbort` |
| `System.Security.Cryptography.CngUIPolicy` | `MayLeakOnAbort` |
| `System.Security.Cryptography.ECDiffieHellman` | `MayLeakOnAbort` |
| `System.Security.Cryptography.ECDiffieHellmanPublicKey` | `MayLeakOnAbort` |
| `System.Security.Cryptography.ECDiffieHellmanCng` | `MayLeakOnAbort` |
| `System.Security.Cryptography.ECDiffieHellmanCngPublicKey` | `MayLeakOnAbort` |
| `System.Security.Cryptography.ECDsa` | `MayLeakOnAbort` |
| `System.Security.Cryptography.ECDsaCng` | `MayLeakOnAbort` |
| `System.Security.Cryptography.ManifestSignatureInformation` | `MayLeakOnAbort` |
| `System.Security.Cryptography.ManifestSignatureInformationCollection` | `MayLeakOnAbort` |
| `System.Security.Cryptography.MD5Cng` | `MayLeakOnAbort` |
| `System.Security.Cryptography.SHA1Cng` | `MayLeakOnAbort` |
| `System.Security.Cryptography.SHA256Cng` | `MayLeakOnAbort` |
| `System.Security.Cryptography.SHA256CryptoServiceProvider` | `MayLeakOnAbort` |
| `System.Security.Cryptography.SHA384Cng` | `MayLeakOnAbort` |
| `System.Security.Cryptography.SHA384CryptoServiceProvider` | `MayLeakOnAbort` |
| `System.Security.Cryptography.SHA512Cng` | `MayLeakOnAbort` |
| `System.Security.Cryptography.SHA512CryptoServiceProvider` | `MayLeakOnAbort` |
| `System.Security.Cryptography.StrongNameSignatureInformation` | `MayLeakOnAbort` |
| `System.Security.Cryptography.X509Certificates.AuthenticodeSignatureInformation` | `MayLeakOnAbort` |
| `System.Security.Cryptography.X509Certificates.TimestampInformation` | `MayLeakOnAbort` |
| `Microsoft.Win32.SafeHandles.SafePipeHandle` | `MayLeakOnAbort` |
| `System.TimeZoneInfo` | `MayLeakOnAbort` |
| `System.TimeZoneNotFoundException` | `MayLeakOnAbort` |
| `System.InvalidTimeZoneException` | `MayLeakOnAbort` |
| `System.Diagnostics.EventSchemaTraceListener` | `MayLeakOnAbort` |
| `System.Diagnostics.UnescapedXmlDiagnosticData` | `MayLeakOnAbort` |
| `System.Diagnostics.PerformanceData.CounterData` | `MayLeakOnAbort` |
| `System.Diagnostics.PerformanceData.CounterSetInstanceCounterDataSet` | `MayLeakOnAbort` |
| `System.Diagnostics.PerformanceData.CounterSet` | `MayLeakOnAbort` |
| `System.Diagnostics.PerformanceData.CounterSetInstance` | `MayLeakOnAbort` |
| `System.Collections.Generic.HashSet`1 | `MayLeakOnAbort` |
| `System.IO.Pipes.PipeStream` | `MayLeakOnAbort` |
| `System.IO.Pipes.AnonymousPipeServerStream` | `MayLeakOnAbort` |
| `System.IO.Pipes.AnonymousPipeClientStream` | `MayLeakOnAbort` |
| `System.IO.Pipes.NamedPipeServerStream` | `MayLeakOnAbort` |
| `System.IO.Pipes.NamedPipeClientStream` | `MayLeakOnAbort` |
| `System.IO.Pipes.PipeAccessRule` | `MayLeakOnAbort` |
| `System.IO.Pipes.PipeAuditRule` | `MayLeakOnAbort` |
| `System.IO.Pipes.PipeSecurity` | `MayLeakOnAbort` |
| `System.Threading.LockRecursionException` | `MayLeakOnAbort` |
| `System.Threading.ReaderWriterLockSlim` | `MayLeakOnAbort` |

## Related content

- [Host protection attributes and CLR integration programming](host-protection-attributes-and-clr-integration-programming.md)
- [Disallowed types and members in Microsoft.VisualBasic.dll](disallowed-types-and-members-in-microsoft-visualbasic-dll.md)
- [Disallowed types and members in mscorlib.dll](disallowed-types-and-members-in-mscorlib-dll.md)
- [Disallowed types and members in System.dll](disallowed-types-and-members-in-system-dll.md)
- [Disallowed types and members in System.Data.dll](disallowed-types-and-members-in-system-data-dll.md)
