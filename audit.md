# Audit

## Description
The **Audit** feature allows to track and to log change in a database or a file that occurred when saving modification in Entity Framework.

```csharp
public EntityContext() : base(FiddleHelper.GetConnectionStringSqlServer())
{
	// Add your configuration here
	var audit = this.Configuration.Audit;			
	audit.AutoSaveSet = this.XmlAuditEntries;
	audit.IsEnabled = true;
}

// ...code...

context.Customers.Add(new Customer() { Name = "Customer_A", Description = "Description" });
context.Customers.Add(new Customer() { Name = "Customer_B", Description = "Description" });			
context.Customers.Add(new Customer() { Name = "Customer_C", Description = "Description" });

// Save changes with Audit Enabled
context.SaveChanges();

// Display Audit Trail
FiddleHelper.WriteTable("Entity Framework - Audit Trail", context.XmlAuditEntries.Where<Customer>().ToList());
```
[Try it](https://dotnetfiddle.net/1kVazO)

This feature allows to handle various scenario such as:
- [Save audit trail in database](#save-audit-trail-in-database)
- [Save audit trail in a file](#save-audit-trail-in-a-file)
- [Save audit trail in a different context](#save-audit-trail-in-a-different-context)
- [Display audit trail history](#display-audit-trail-history)

### What is supported?
- `SaveChanges()`
- `BatchSaveChanges()`
- `BulkSaveChanges()`
- All types of modifications:
   - `EntityAdded`
   - `EntityModified`
   - `EntityDeleted`
   - `EntitySoftDeleted`
   - `RelationshipAdded`
   - `RelationshipDeleted`

### Advantage
- Track what, who, and when a value changed
- Keep the history (audit trail) of all changes
- Show to the user the history (audit trail) of all changes

## Getting Started

### Enable Auditing
By default, to not impact performance, the **Audit** feature is disabled. You can activate it by enabling the following configuration `context.Configuration.Audit.IsEnabled = true`.

#### Always enabled
You can have the **Audit** feature always activated by enabling it in your context constructor.

```csharp
// ...code...
```
[Try it](https://dotnetfiddle.net/gmlMIz)

#### On Demand enabled
You can activate the **Audit** feature on demand by enabling it after the context is created.

```csharp
// ...code...
```
[Try it](https://dotnetfiddle.net/ewsolr)

### AutoSave Set
To automatically save changes in your database, you need to specify the set in which audit entries will be added then saved.

One of those following set must be added to your context:
- `DbSet<AuditEntry>` (Best for viewing, worse for insert)
- `DbSet<XmlAuditEntry>`
- `DbSet<JsonAuditEntry>`
- `DbSet<IAuditEntry>` (or a class that inherit from `IAuditEntry`)

```csharp
// ...code...
```
[Try it](https://dotnetfiddle.net/2HLtF4)

### AutoSave Action
To automatically save changes in another context or a file, you need to specify an AutoSaveAction that will always be executed after a save is completed.

```csharp
public EntityContext() : base(FiddleHelper.GetConnectionStringSqlServer())
{
	var audit = this.Configuration.Audit;			
	audit.AutoSaveAction = (context, auditing) => {
		foreach(var entry in auditing.EntriesXml)
		{
			Log.AppendLine("EntitySetName: " + entry.EntitySetName);
			Log.AppendLine("EntityTypeName: " + entry.EntityTypeName);
			Log.AppendLine("State: " + entry.State);
			Log.AppendLine("StateName: " + entry.StateName);
			Log.AppendLine("CreatedBy: " + entry.CreatedBy);
			Log.AppendLine("CreatedDate: " + entry.CreatedDate);
			Log.AppendLine("XmlProperties: " + entry.XmlProperties);
			Log.AppendLine("---");
			Log.AppendLine("---");
			Log.AppendLine("---");
		}				
	};
	audit.IsEnabled = true;
}

// ...code...

using (var context = new EntityContext())
{
	context.Customers.Add(new Customer() { Name = "Customer_A", Description = "Description" });
	context.Customers.Add(new Customer() { Name = "Customer_B", Description = "Description" });			
	context.Customers.Add(new Customer() { Name = "Customer_C", Description = "Description" });

	// Save changes with Audit Enabled
	context.SaveChanges();
}

// Display Audit Trail
Console.WriteLine(Log.ToString());
```
[Try it](https://dotnetfiddle.net/s7Zk45)

### Save properties as a table row
To save properties (one row per property), you need to use the `AuditEntry` and `AuditEntryProperty` class. Only the `AuditEntry` need to be specified in the `AutoSaveSet` audit.

This format is very useful when looking at your properties but **NOT RECOMMENDED**.  Since one insert will be performed for every property, you may severely impact your application performance. We highly recommend using `XmlAuditEntry` or `JsonAuditEntry` instead.

```csharp
// ...code...
```
[Try it](https://dotnetfiddle.net/lQIzmq)

### Save properties as XML
The `XmlAuditEntry` class allow you to save your properties in an XML format.

```csharp
// ...code...
```
[Try it](https://dotnetfiddle.net/cLLDW7)

### Save properties as Json
The `JsonAuditEntry` class allow you to save your properties in a Json format.

```csharp
// ...code...
```
[Try it](https://dotnetfiddle.net/AZqGdO)

### Save custom audit
The `IAuditEntry` interface allows creating a custom audit class by inheriting from the interface. Properties are populated by reflection when the name matches ones that are supported. 

See [supported properties name](#iauditentry).

```csharp
// ...code...
```
[Try it](https://dotnetfiddle.net/lVIW2I)

## Real Life Scenarios

### Save audit trail in a database
You have a requirement to track and log all changes in a database.

```csharp
// ...code...
```
[Try it**](https://dotnetfiddle.net/b1kwHs)

### Save audit trail in a file
You have a requirement to track and log all changes in a file.

```csharp
// ...code...
```
[Try it**](https://dotnetfiddle.net/b1kwHs)

### Save audit trail in a different context
You have a requirement to track and log all changes in a different context or database.

```csharp
public EntityContext() : base(FiddleHelper.GetConnectionStringSqlServer())
{
	// Add your configuration here
	var audit = this.Configuration.Audit;			
	audit.AutoSaveAction = (context, auditing) => {
		var auditContext = new AuditContext();
		auditContext.XmlAuditEntries.AddRange(auditing.EntriesXml);
		auditContext.SaveChanges();
	};
	
	audit.IsEnabled = true;
}

// ...code...

using (var context = new EntityContext())
{
	context.Customers.Add(new Customer() { Name = "Customer_A", Description = "Description" });
	context.Customers.Add(new Customer() { Name = "Customer_B", Description = "Description" });			
	context.Customers.Add(new Customer() { Name = "Customer_C", Description = "Description" });
	
	// Save changes with Audit Enabled
	context.SaveChanges();
}

using(var context = new AuditContext())
{
	// Display Audit Trail
	FiddleHelper.WriteTable("Entity Framework - Audit Trail", context.XmlAuditEntries.ToList());
}
```
[Try it](https://dotnetfiddle.net/JMVLsE)

### Display audit trail history
You have a requirement to display in the application all the changes that have been made on a specific entity

```csharp
// ...code...
```
[Try it**](https://dotnetfiddle.net/b1kwHs)

## Documentation

### AuditManager

The `AuditManager` allow you to configure all options for the **Audit**.

###### Properties

| Name | Description | Default | Example |
| :--- | :---------- | :-----: | :------ |
| `IsEnabled` | Gets or sets if the `Audit` feature is enabled. By default, this feature is disabled to not impact the performance. | `false` | [Try it](https://dotnetfiddle.net/4xrM1d) |
| `AutoSaveAction` | Gets or sets the `AutoSaveAction`. This option is usually used to save audit log in a file or different context. | `null` | [Try it](https://dotnetfiddle.net/YzYyE7) |
| `AutoSaveSet` | Gets or sets the `AutoSaveSet`. This option is usually used to auto save audit log in a database. | `null` | [Try it](https://dotnetfiddle.net/hXzGVu) |
| `LastAudit` | Gets the last `Audit` log. This option give you all audit information after a save is performed. | `null` | [Try it](https://dotnetfiddle.net/8ce2Y1) |

###### Methods (Display)

| Name | Description | Example |
| :--- | :---------- | :------ |
| `DisplayDBNullAsNull(bool value)` | Format DBNull.Value as 'null' value. True by default. | [Try it](https://dotnetfiddle.net/8dawtN) |
| `DisplayFormatValue<TEntityType>(Expression<Func<TEntityType, object>> propertySelector, Func<TEntityType, object, object> formatter)` | Formats value for selected properties from entities of 'TEntityType' type. | [Try it**](https://dotnetfiddle.net/dBOdw2) |
| `DisplayNameEntity<TEntityType>(Func<TEntityType, string, string>)` formatter | Formats entity name for selected properties from entities of 'TEntityType' type. | [Try it**](https://dotnetfiddle.net/q7T7nl)  |
| `DisplayNameProperty<TEntityType>(Expression<Func<TEntityType, object>> propertySelector, Func<TEntityType, string, string> formatter)` | Formats property name for selected properties from entities of 'TEntityType' type. | [Try it**](https://dotnetfiddle.net/Zoric3)  |

###### Methods (Include & Exclude)

| Name | Description | Example |
| :--- | :---------- | :------ |
| `ExcludeEntity()` | Excludes from the audit all entities. | [Try it](https://dotnetfiddle.net/MrWtV4) |
| `ExcludeEntity<TEntityType>()` | Excludes from the audit all entities of type 'TEntityType' type. | [Try it](https://dotnetfiddle.net/Nes7be) |
| `ExcludeEntity(Func<object, bool> predicate)` | Excludes from the audit all entities which satisfy the predicate. | [Try it](https://dotnetfiddle.net/tWTkaC)  |
| `ExcludeEntity(AuditEntryState entryState)` | Excludes from the audit all entities for the specified entry state. | [Try it](https://dotnetfiddle.net/vLHfk9)  |
| `ExcludeEntity<TEntityType>(AuditEntryState entryState)` | Excludes from the audit all entities of type 'TEntityType' type for the specified entry state. | [Try it](https://dotnetfiddle.net/g5t6FB) |
| `ExcludeEntity(Func<object, bool> predicate, AuditEntryState entryState)` | Excludes from the audit all entities which satisfy the predicate for the specified entry state. | [Try it](https://dotnetfiddle.net/g22Zlv) |
| `ExcludeProperty()` | Excludes from the audit all properties. Key properties are never excluded.| [Try it**](https://dotnetfiddle.net/dBOdw2) |
| `ExcludeProperty<TEntityType>()` | Excludes from the audit all properties from entities of 'TEntityType' type. Key properties are never excluded. | [Try it**](https://dotnetfiddle.net/q7T7nl)  |
| `ExcludeProperty<TEntityType>(Expression<Func<TEntityType, object>> propertySelector)` | Excludes from the audit selected properties from entities of 'TEntityType' type. Key properties are never excluded. | [Try it**](https://dotnetfiddle.net/Zoric3)  |
| `ExcludePropertyUnchanged()` | Excludes from the audit all properties unchanged. Key properties are never excluded. | [Try it**](https://dotnetfiddle.net/2IBfGq) |
| `ExcludePropertyUnchanged(Func<object, bool> predicate)` | Excludes from the audit all properties unchanged from entities of type 'TEntityType' type. Key properties are never excluded. | [Try it**](https://dotnetfiddle.net/lqfF8b) |
| `ExcludePropertyUnchanged<TEntityType>()` | Excludes from the audit all properties unchanged which satisfy the predicate. Key properties are never excluded. | [Try it**](https://dotnetfiddle.net/dBOdw2) |
| `IncludeEntity()` | Includes in the the audit all entities. | [Try it](https://dotnetfiddle.net/r0Rq73) |
| `IncludeEntity<TEntityType>()` | Includes in the audit all entities of 'TEntityType' type. | [Try it](https://dotnetfiddle.net/rjhUQb) |
| `IncludeEntity(Func<object, bool> predicate)` | Includes in the audit all entities which satisfy the predicate. | [Try it](https://dotnetfiddle.net/TdqHmK)  |
| `IncludeEntity(AuditEntryState entryState)` | Includes in the the audit all entities for the specified entry state. | [Try it](https://dotnetfiddle.net/YA3Gr0)  |
| `IncludeEntity<TEntityType>(AuditEntryState entryState)` | Includes in the audit all entities of 'TEntityType' for the specified entry state. | [Try it](https://dotnetfiddle.net/PFY5Pp) |
| `IncludeEntity(Func<object, bool> predicate, AuditEntryState entryState)` | Includes in the audit all entities which satisfy the predicate for the specified entry state. | [Try it](https://dotnetfiddle.net/yPchrX) |
| `IncludeProperty()` | Includes in the audit all properties. | [Try it**](https://dotnetfiddle.net/dBOdw2) |
| `IncludeProperty<TEntityType>()` | Includes in the audit all properties from entities of 'TEntityType' type. | [Try it**](https://dotnetfiddle.net/q7T7nl)  |
| `IncludeProperty<TEntityType>(Expression<Func<TEntityType, object>> propertySelector)` | Includes in the audit selected properties from entities of 'TEntityType' type. | [Try it**](https://dotnetfiddle.net/Zoric3)  |
| `IncludePropertyUnchanged()` | Includes to the audit all properties unchanged. | [Try it**](https://dotnetfiddle.net/2IBfGq) |
| `IncludePropertyUnchanged(Func<object, bool> predicate)` | Includes to the audit all properties unchanged from entities of type 'TEntityType' type. | [Try it**](https://dotnetfiddle.net/lqfF8b) |
| `IncludePropertyUnchanged<TEntityType>()` | Includes to the audit all properties unchanged which satisfy the predicate. | [Try it**](https://dotnetfiddle.net/dBOdw2) |

###### Methods (Soft Delete)

| Name | Description | Example |
| :--- | :---------- | :------ |
| `SoftDeleted(Func<object, bool> predicate)` | Change the AuditEntryState from "EntityModified' to "EntitySoftDeleted" for all entities which satisfy the soft delete predicate. | [Try it](https://dotnetfiddle.net/k1h8fr) |
| `SoftDeleted<TEntityType>(Func<TEntityType, bool> predicate)` | Change the AuditEntryState from "EntityModified" to "EntitySoftDeleted" for all entities of 'TEntityType' type and which satisfy the soft delete predicate. | [Try it](https://dotnetfiddle.net/OdVFGF) |

### Audit

The `Audit` class provide information about the audit trail.

###### Properties

| Name | Description | Example |
| :--- | :---------- | :------ |
| `Entries` | Gets or sets the audit entries. | [Try it](https://dotnetfiddle.net/In5zMB) |
| `EntriesJson` | Gets the audit entries with properties as Json. | [Try it](https://dotnetfiddle.net/1Pt2Ny) |
| `EntriesXml` | Gets the audit entries with properties as Xml. | [Try it](https://dotnetfiddle.net/9WKMvp) |
| `Manager` | Gets the audit manager. | [Try it](https://dotnetfiddle.net/LVU9UD) |

###### Methods
| Name | Description | Example |
| :--- | :---------- | :------ |
| `EntriesAs<T>() where T : IAuditEntry` | Gets customs audit entries that implement the interface `IAuditEntry` | [Try it**](https://dotnetfiddle.net/lqfF8b) |

### AuditEntry

The `AuditEntry` class contains information about the entry and a list of `AuditEntryProperty`.

###### Properties (Mapped)

| Name | Description | Example |
| :--- | :---------- | :------ |
| `AuditEntryID` | Gets or sets the `AuditEntryID`. | [Try it](https://dotnetfiddle.net/bn3OpH) |
| `EntitySetName` | Gets or sets the `EntitySet` name. | [Try it](https://dotnetfiddle.net/wEjMFB) |
| `EntityTypeName` | Gets or sets the `EntityType` name. | [Try it](https://dotnetfiddle.net/Ulretn) |
| `State` | Gets or sets the `AuditEntryState`. | [Try it](https://dotnetfiddle.net/hitoCH) |
| `StateName` | Gets or sets the `AuditEntryState` name. | [Try it](https://dotnetfiddle.net/hzefDr) |
| `CreatedBy` | Gets or sets the `AuditEntry` created user. | [Try it](https://dotnetfiddle.net/JjBEqS) |
| `CreatedDate` | Gets or sets the `AuditEntry` created date. | [Try it](https://dotnetfiddle.net/XG8s4n) |
| `Properties` | Gets or sets the `AuditEntry` properties. | [Try it](https://dotnetfiddle.net/FYsJgt) |

###### Properties (Unmapped)

| Name | Description | Example |
| :--- | :---------- | :------ |
| `ParentAudit` | Gets or sets the parent `Audit`. | [Try it**](https://dotnetfiddle.net/28AdvH) |
| `Entity` | Gets or sets the audited `Entity`. | [Try it**](https://dotnetfiddle.net/8z8spq) |
| `Entry` | Gets or sets the audited `ObjectStateEntry`.  | [Try it**](https://dotnetfiddle.net/DP6Del) |

<details>
  <summary>Database First SQL</summary>

```sql
SELECT 1
```
</details>

### AuditEntryProperty

The `AuditEntryProperty` contains information about `property`.

###### Properties

| Name | Description | Example |
| :--- | :---------- | :------ |
| `Parent` | Gets or sets the parent `AuditEntry`. | [Try it](https://dotnetfiddle.net/WXpAtA) |
| `AuditEntryPropertyID` | Gets or sets the `AuditEntryPropertyID`. | [Try it](https://dotnetfiddle.net/C8GBWu) |
| `AuditEntryID` | Gets or sets the `AuditEntryID`. | [Try it](https://dotnetfiddle.net/kJzQ6i) |
| `RelationName` | Gets or sets the relation name. Only available for `RelationshipAdded` and `RelationshipDeleted` state | [Try it](https://dotnetfiddle.net/LTd809) |
| `PropertyName` | Gets or sets the property name. | [Try it](https://dotnetfiddle.net/oYqqV0) |
| `OldValue` | Gets or sets the old value formatted as string. Avalable for `Modified`, `Deleted`, and `RelationshipDeleted` state. | [Try it](https://dotnetfiddle.net/hNUCx6) |
| `NewValue` | Gets or sets the new value formatted as string. Avalable for `Insert`, `Modified`, and `RelationshipModified` state. | [Try it](https://dotnetfiddle.net/lrGw9a) |

###### Properties (Unmapped)

| Name | Description | Example |
| :--- | :---------- | :------ |
| `OldValueRaw` | Gets or sets the old raw value. This is the original raw value without being formatted. | [Try it](https://dotnetfiddle.net/cxK2Hq) |
| `NewValueRaw` | Gets or sets the new raw value. This is the original raw value without being formatted. | [Try it](https://dotnetfiddle.net/zetqj9) |

<details>
  <summary>Database First SQL</summary>

```sql
SELECT 1
```
</details>

### JsonAuditEntry

The `JsonAuditEntry` class contains information about the `AuditEntry` and all properties saved in a `Json` format.

###### Properties (Mapped)

| Name | Description | Example |
| :--- | :---------- | :------ |
| `JsonAuditEntryID` | Gets or sets the `AuditEntryID`. | [Try it](https://dotnetfiddle.net/KNnZ7i) |
| `EntitySetName` | Gets or sets the `EntitySet` name. | [Try it](https://dotnetfiddle.net/u7BXwl) |
| `EntityTypeName` | Gets or sets the `EntityType` name. | [Try it](https://dotnetfiddle.net/iVGV38) |
| `State` | Gets or sets the `AuditEntryState`. | [Try it](https://dotnetfiddle.net/5Y7rqE) |
| `StateName` | Gets or sets the `AuditEntryState` name. | [Try it](https://dotnetfiddle.net/Gzu8f0) |
| `CreatedBy` | Gets or sets the `AuditEntry` created user. | [Try it](https://dotnetfiddle.net/en1Sd1) |
| `CreatedDate` | Gets or sets the `AuditEntry` created date. | [Try it](https://dotnetfiddle.net/q9GA7E) |
| `JsonProperties` | Gets or sets audit properties formatted as `Json`. | [Try it](https://dotnetfiddle.net/YMiRH6) |

<details>
  <summary>Database First SQL</summary>

```sql
SELECT 1
```
</details>

### XmlAuditEntry

The `XmlAuditEntry` class contains information about the `AuditEntry` and all properties saved in a `Xml` format.

###### Properties (Mapped)

| Name | Description | Example |
| :--- | :---------- | :------ |
| `XmlAuditEntryID` | Gets or sets the `AuditEntryID`. | [Try it](https://dotnetfiddle.net/ohl81V) |
| `EntitySetName` | Gets or sets the `EntitySet` name. | [Try it](https://dotnetfiddle.net/r1sti8) |
| `EntityTypeName` | Gets or sets the `EntityType` name. | [Try it](https://dotnetfiddle.net/InPE7m) |
| `State` | Gets or sets the `AuditEntryState`. | [Try it](https://dotnetfiddle.net/yl2ESG) |
| `StateName` | Gets or sets the `AuditEntryState` name. | [Try it](https://dotnetfiddle.net/Gok4r5) |
| `CreatedBy` | Gets or sets the `AuditEntry` created user. | [Try it](https://dotnetfiddle.net/SooSeu) |
| `CreatedDate` | Gets or sets the `AuditEntry` created date. | [Try it](https://dotnetfiddle.net/LT6aSE) |
| `XmlProperties` | Gets or sets audit properties formatted as `Xml`. | [Try it](https://dotnetfiddle.net/Dcldvf) |

<details>
  <summary>Database First SQL</summary>

```sql
SELECT 1
```
</details>

### IAuditEntry

The `IAuditEntry` interface doesn't have any properties. However, properties are populated via `reflection` when a property with the same name is implemented.

For example, if the `class` that inherit from `IAuditEntry` has a property named `EntitySetName`, the properties will automatically be populated with the right value.

###### Properties

| Name | Description | Example |
| :--- | :---------- | :------ |
| `AuditEntryID` | Gets or sets the `AuditEntryID`. | [Try it**](https://dotnetfiddle.net/8z8spq) |
| `EntitySetName` | Gets or sets the `EntitySet` name. | [Try it**](https://dotnetfiddle.net/DP6Del) |
| `EntityTypeName` | Gets or sets the `EntityType` name. | [Try it**](https://dotnetfiddle.net/28AdvH) |
| `State` | Gets or sets the `AuditEntryState`. | [Try it**](https://dotnetfiddle.net/28AdvH) |
| `StateName` | Gets or sets the `AuditEntryState` name. | [Try it**](https://dotnetfiddle.net/28AdvH) |
| `CreatedBy` | Gets or sets the `AuditEntry` created user. | [Try it**](https://dotnetfiddle.net/28AdvH) |
| `CreatedDate` | Gets or sets the `AuditEntry` created date. | [Try it**](https://dotnetfiddle.net/28AdvH) |
| `JsonProperties` | Gets or sets audit properties formatted as `Json`. | [Try it**](https://dotnetfiddle.net/28AdvH) |
| `XmlProperties` | Gets or sets audit properties formatted as `Xml`. | [Try it**](https://dotnetfiddle.net/28AdvH) |

## Limitations

### Bulk Operations & Batch Operations
All operations that doesn't use the `ChangeTracker` (required to create the audit trail) are currently not supported:
- [Bulk Insert](/bulk-insert)
- [Bulk Update](/bulk-update)
- [Bulk Delete](/bulk-delete)
- [Bulk Merge](/bulk-merge)
- [Bulk Synchronize](/bulk-synchronize)
- [Delete from Query](/delete-from-query)
- [Update from Query](/update-from-query)
