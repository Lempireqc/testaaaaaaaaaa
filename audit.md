# Audit

## Description
The **EF Audit** feature allows you to create an audit trail of all changes that occured when saving in Entity Framework. 

The audit trail can be automatically saved in a database or log file.

```csharp
public class EntityContext : DbContext
{
	public EntityContext() : base(FiddleHelper.GetConnectionStringSqlServer())
	{
		// Add your configuration here
		this.Configuration.Audit.AutoSaveSet = this.XmlAuditEntries;
		this.Configuration.Audit.IsEnabled = true;
	}
	
	public DbSet<Customer> Customers { get; set; }
	public DbSet<XmlAuditEntry> XmlAuditEntries { get; set; }
}

public static void Main()
{
	using (var context = new EntityContext())
	{
		context.Customers.Add(new Customer() { Name = "Customer_A", Description = "Description" });
		
		// Save changes and Audit trail in database
		context.SaveChanges();
		
		// Display Audit trail
		var auditEntries = context.XmlAuditEntries.ToAuditEntries();
		FiddleHelper.WriteTable("1 - Audit Entries", auditEntries);
		FiddleHelper.WriteTable("2 - Audit Properties", auditEntries.SelectMany(x => x.Properties));
	}
}
```
[Try it](https://dotnetfiddle.net/1kVazO)

This feature allows you to handle various scenario such as:
- [Saving audit trail in a database](#saving-audit-trail-in-a-database)
- [Saving audit trail in a log file](#saving-audit-trail-in-a-log-file)
- [Saving audit trail in a different database](#saving-audit-trail-in-a-different-database)
- [Displaying audit trail history](#displaying-audit-trail-history)

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
- Track what, who, and when a value is changed
- Keep the audit trail of all changes
- Display the audit trail of all changes

### Community vs Enterprise
The `Audit` feature is free to use in the **Community** version.

The **Enterprise** version offer performance enhancement by automatically saving audit entries using the `BulkInsert`.

## Getting Started

### Enable Auditing
By default, not to impact performance, the **Audit** feature is disabled. You can activate it by enabling the following configuration `context.Configuration.Audit.IsEnabled = true`.

#### Always enabled
To have the **Audit** feature always enabled, you activate it in your context constructor.

```csharp
public class EntityContext : DbContext
{
	public EntityContext() : base(FiddleHelper.GetConnectionStringSqlServer())
	{
		// Add your configuration here
		this.Configuration.Audit.AutoSaveSet = this.XmlAuditEntries;
		this.Configuration.Audit.IsEnabled = true;
	}
	
	public DbSet<Customer> Customers { get; set; }
	public DbSet<XmlAuditEntry> XmlAuditEntries { get; set; }
}

public static void Main()
{
	using (var context = new EntityContext())
	{
		context.Customers.Add(new Customer() { Name = "Customer_A", Description = "Description" });
		
		// Save changes and Audit trail in database
		context.SaveChanges();
		
		// Display Audit trail
		var auditEntries = context.XmlAuditEntries.ToAuditEntries();
		FiddleHelper.WriteTable("1 - Audit Entries", auditEntries);
		FiddleHelper.WriteTable("2 - Audit Properties", auditEntries.SelectMany(x => x.Properties));
	}
}
```
[Try it](https://dotnetfiddle.net/gmlMIz)

#### On Demand enabled
To have the **Audit** feature on demand enabled, you activate it after the context is created.

```csharp
public class EntityContext : DbContext
{
	public EntityContext() : base(FiddleHelper.GetConnectionStringSqlServer())
	{
		// Add your configuration here
		this.Configuration.Audit.AutoSaveSet = this.XmlAuditEntries;
	}
	
	public DbSet<Customer> Customers { get; set; }
	public DbSet<XmlAuditEntry> XmlAuditEntries { get; set; }
}

public static void Main()
{
	using (var context = new EntityContext())
	{
		// You can activate the Audit feature on demand by enabling it after the context is created.
		context.Configuration.Audit.IsEnabled = true;
		
		context.Customers.Add(new Customer() { Name = "Customer_A", Description = "Description" });
		
		// Save changes and Audit trail in database
		context.SaveChanges();
		
		// Display Audit trail
		var auditEntries = context.XmlAuditEntries.ToAuditEntries();
		FiddleHelper.WriteTable("1 - Audit Entries", auditEntries);
		FiddleHelper.WriteTable("2 - Audit Properties", auditEntries.SelectMany(x => x.Properties));
	}
}
```
[Try it](https://dotnetfiddle.net/ewsolr)

### Last Audit
The latest audit can be accessed with the `LastAudit` property.

The `LastAudit` property give you additional information that are not saved such as:
- Entity
- Entry
- OldValueRaw
- NewValueRaw

```csharp
// ...code...
```
[Try it](https://dotnetfiddle.net/qoNEMi)

### AutoSave Set
To automatically save the audit trail in your database, you need to specify the `DbSet<>` in which audit entries will be added then saved.

One of those following set must be added to your context:

| Name | Description | Example |
| :--- | :---------- | :------ |
| `AuditEntry` | The `AuditEntry` class allow you to save one row per property (Only recommand for `Enterprise` version). The `DbSet<AuditEntry>` and `DbSet<AuditEntryProperty>` must be added to your context. | [Try it**](https://dotnetfiddle.net/SooSeu) |
| `XmlAuditEntry` | The `XmlAuditEntry` class allow you to save your properties in an XML format. The `DbSet<XmlAuditEntry>` must be added to your context. | [Try it](https://dotnetfiddle.net/2HLtF4) |

### AutoSave Action
To automatically save the audit trail in another context or a log file, you need to specify an `AutoSaveAction`. This action will be executed after all saves are completed.

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

## Real Life Scenarios

### Saving audit trail in a database
Your application need to keep an audit trail of all changes in a database. You can automatically save the audit trail by specifying an `AutoSaveSet`.

```csharp
// ...code...
```
[Try it](https://dotnetfiddle.net/8fBiZj)

### Saving audit trail in a log file
Your application need to keep an audit trail of all changes in a log file. You can automatically save the audit trail in a log file by specifying an `AutoSaveAction`.


```csharp
// ...code...
```
[Try it](https://dotnetfiddle.net/1JvBQ8)

### Saving audit trail in a different database
Your application need to keep an audit trail of all changes in a different database. You can automatically save the audit trail in a different database file by specifying an `AutoSaveAction`.

```csharp
public class EntityContext : DbContext
{
	public EntityContext() : base(FiddleHelper.GetConnectionStringSqlServer())
	{
		// Add your configuration here			
		this.Configuration.Audit.AutoSaveAction = (context, auditing) => {
			var auditContext = new AuditContext();
			auditContext.XmlAuditEntries.AddRange(auditing.EntriesXml);
			auditContext.SaveChanges();
		};
		
		this.Configuration.Audit.IsEnabled = true;
	}
	
	public DbSet<Customer> Customers { get; set; }
}

public class AuditContext : DbContext
{
	public AuditContext() : base(FiddleHelper.GetConnectionStringSqlServer())
	{
	}

	public DbSet<XmlAuditEntry> XmlAuditEntries { get; set; }
}

public static void Main()
{
	using (var context = new EntityContext())
	{
		context.Customers.Add(new Customer() { Name = "Customer_A", Description = "Description" });
		
		// Save changes and Audit trail in database
		context.SaveChanges();
	}
	
	using(var auditContext = new AuditContext())
	{
		// Display Audit trail
		var auditEntries = auditContext.XmlAuditEntries.ToAuditEntries();
		FiddleHelper.WriteTable("1 - Audit Entries", auditEntries);
		FiddleHelper.WriteTable("2 - Audit Properties", auditEntries.SelectMany(x => x.Properties));
	}
}
```
[Try it](https://dotnetfiddle.net/JMVLsE)

### Displaying audit trail history
Your application need to display the audit trail of all changes in an user interface. If your audit trail is saved in a database, you can retrieve and display audit entries by using your audit `DbSet<>`.

```csharp
// ...code...
```
[Try it](https://dotnetfiddle.net/wfoPB1)

## Documentation

### AuditManager

The `AuditManager` allow you to configure how the audit trail will be created, saved, and retrieved.

###### Properties

| Name | Description | Default | Example |
| :--- | :---------- | :-----: | :------ |
| `IsEnabled` | Gets or sets if the `Audit` feature is enabled. By default, this feature is disabled not to impact the performance. | `false` | [Try it](https://dotnetfiddle.net/4xrM1d) |
| `AutoSaveAction` | Gets or sets the `AutoSaveAction`. This option is usually used to automatically save the audit trail in a log file or different database. | `null` | [Try it](https://dotnetfiddle.net/YzYyE7) |
| `AutoSaveSet` | Gets or sets the `AutoSaveSet`. This option is usually used to automatically save the audit trail in a database. | `null` | [Try it](https://dotnetfiddle.net/hXzGVu) |
| `LastAudit` | Gets the last `Audit` trail. | `null` | [Try it](https://dotnetfiddle.net/8ce2Y1) |

###### Methods (Display)

| Name | Description | Example |
| :--- | :---------- | :------ |
| `DisplayDBNullAsNull(bool value)` | Format DBNull.Value as 'null' value. True by default. | [Try it](https://dotnetfiddle.net/8dawtN) |
| `DisplayFormatType<TEntityType>(Func<TEntityType, string> formatter)` | Format the specified type into a string. | [Try it**](https://dotnetfiddle.net/dBOdw2) |
| `DisplayEntityName<TEntityType>(string name)` | Formats entity name for selected properties from entities of 'TEntityType' type. | [Try it**](https://dotnetfiddle.net/q7T7nl)  |
| `DisplayEntityName<TEntityType>(Func<TEntityType, string, string> formatter)` | Formats entity name for selected properties from entities of 'TEntityType' type. | [Try it**](https://dotnetfiddle.net/q7T7nl)  |
| `DisplayPropertyName<TEntityType>(Expression<Func<TEntityType, object>> propertySelector, string name` | Specify the property name for the specifed property of the 'TEntityType' type. | [Try it**](https://dotnetfiddle.net/Zoric3)  |
| `DisplayPropertyName<TEntityType>(Expression<Func<TEntityType, object>> propertySelector, Func<TEntityType, string, string> formatter)` | Formats property name for selected properties from entities of 'TEntityType' type. | [Try it**](https://dotnetfiddle.net/Zoric3)  |

###### Methods (Include & Exclude)

| Name | Description | Example |
| :--- | :---------- | :------ |
| `ExcludeEntity()` | Excludes from the audit all entities. | [Try it](https://dotnetfiddle.net/MrWtV4) |
| `ExcludeEntity<TEntityType>()` | Excludes from the audit all entities of 'TEntityType' type. | [Try it](https://dotnetfiddle.net/Nes7be) |
| `ExcludeEntity(Func<object, bool> predicate)` | Excludes from the audit all entities which satisfy the predicate. | [Try it](https://dotnetfiddle.net/tWTkaC)  |
| `ExcludeEntity(AuditEntryState entryState)` | Excludes from the audit all entities for the specified entry state. | [Try it](https://dotnetfiddle.net/vLHfk9)  |
| `ExcludeEntity<TEntityType>(AuditEntryState entryState)` | Excludes from the audit all entities of 'TEntityType' type for the specified entry state. | [Try it](https://dotnetfiddle.net/g5t6FB) |
| `ExcludeEntity(Func<object, bool> predicate, AuditEntryState entryState)` | Excludes from the audit all entities which satisfy the predicate for the specified entry state. | [Try it](https://dotnetfiddle.net/g22Zlv) |
| `ExcludeEntity<TEntityType>(Func<TEntityType, bool> predicate, AuditEntryState entryState)` | Excludes from the audit all entities which satisfy the predicate for the specified entry state. | [Try it](https://dotnetfiddle.net/g22Zlv) |
| `ExcludeProperty()` | Excludes from the audit all properties. Key properties are never excluded.| [Try it](https://dotnetfiddle.net/2KULLj) |
| `ExcludeProperty<TEntityType>()` | Excludes from the audit all properties from entities of 'TEntityType' type. Key properties are never excluded. | [Try it](https://dotnetfiddle.net/5XTmn4)  |
| `ExcludeProperty<TEntityType>(Expression<Func<TEntityType, object>> propertySelector)` | Excludes from the audit selected properties from entities of 'TEntityType' type. Key properties are never excluded. | [Try it](https://dotnetfiddle.net/BlIDQY)  |
| `ExcludePropertyUnchanged()` | Excludes from the audit all properties unchanged. Key properties are never excluded. | [Try it](https://dotnetfiddle.net/9Aw0vq) |
| `ExcludePropertyUnchanged(Func<object, bool> predicate)` | Excludes from the audit all properties unchanged which satisfy the predicate. Key properties are never excluded. | [Try it](https://dotnetfiddle.net/qhTX5h) |
| `ExcludePropertyUnchanged<TEntityType>()` | Excludes from the audit all properties unchanged from entities of type 'TEntityType' type. Key properties are never excluded. | [Try it](https://dotnetfiddle.net/EKOk3a) |
| `IncludeEntity()` | Includes in the the audit all entities. | [Try it](https://dotnetfiddle.net/r0Rq73) |
| `IncludeEntity<TEntityType>()` | Includes in the audit all entities of 'TEntityType' type. | [Try it](https://dotnetfiddle.net/rjhUQb) |
| `IncludeEntity(Func<object, bool> predicate)` | Includes in the audit all entities which satisfy the predicate. | [Try it](https://dotnetfiddle.net/TdqHmK)  |
| `IncludeEntity(AuditEntryState entryState)` | Includes in the the audit all entities for the specified entry state. | [Try it](https://dotnetfiddle.net/YA3Gr0)  |
| `IncludeEntity<TEntityType>(AuditEntryState entryState)` | Includes in the audit all entities of 'TEntityType' for the specified entry state. | [Try it](https://dotnetfiddle.net/PFY5Pp) |
| `IncludeEntity(Func<object, bool> predicate, AuditEntryState entryState)` | Includes in the audit all entities which satisfy the predicate for the specified entry state. | [Try it](https://dotnetfiddle.net/yPchrX) |
| `IncludeEntity<TEntityType>(Func<TEntityType, bool> predicate, AuditEntryState entryState)` | Includes in the audit all entities which satisfy the predicate for the specified entry state. | [Try it](https://dotnetfiddle.net/yPchrX) |
| `IncludeProperty()` | Includes in the audit all properties. | [Try it](https://dotnetfiddle.net/CN9Zq1) |
| `IncludeProperty<TEntityType>()` | Includes in the audit all properties from entities of 'TEntityType' type. | [Try it](https://dotnetfiddle.net/e74jy7)  |
| `IncludeProperty<TEntityType>(Expression<Func<TEntityType, object>> propertySelector)` | Includes in the audit selected properties from entities of 'TEntityType' type. | [Try it](https://dotnetfiddle.net/c7YrfX)  |
| `IncludePropertyUnchanged()` | Includes to the audit all properties unchanged. | [Try it](https://dotnetfiddle.net/RPfZEl) |
| `IncludePropertyUnchanged(Func<object, bool> predicate)` | Includes to the audit all properties unchanged which satisfy the predicate. | [Try it](https://dotnetfiddle.net/HpJ5mz) |
| `IncludePropertyUnchanged<TEntityType>()` | Includes to the audit all properties unchanged from entities of type 'TEntityType' type. | [Try it](https://dotnetfiddle.net/bYmNVq) |

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

This properties values are saved in a database or log file.

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

This properties values are only accessible via the `LastAudit` property.

| Name | Description | Example |
| :--- | :---------- | :------ |
| `ParentAudit` | Gets or sets the parent `Audit`. | [Try it**](https://dotnetfiddle.net/28AdvH) |
| `Entity` | Gets or sets the audited `Entity`. | [Try it**](https://dotnetfiddle.net/8z8spq) |
| `Entry` | Gets or sets the audited `ObjectStateEntry`.  | [Try it**](https://dotnetfiddle.net/DP6Del) |

<details>
  <summary>Database First SQL</summary>

```sql
Coming soon...
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
Coming soon...
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
Coming soon...
```
</details>

### Data Annotations

###### Entity
| Name | Description | Example |
| :--- | :---------- | :------ |
| `AuditDisplay(string name)` | Attribute to change the Audit entity or property display. | [Try it**](https://dotnetfiddle.net/SooSeu) |
| `AuditExclude` | Attribute to exclude from the audit the entity or property. | [Try it**](https://dotnetfiddle.net/LT6aSE) |
| `AuditInclude` | Attribute to include in the audit the entity or property. | [Try it**](https://dotnetfiddle.net/Dcldvf) |


###### Property
| Name | Description | Example |
| :--- | :---------- | :------ |
| `AuditDisplay(string name)` | Attribute to change the Audit entity or property display. | [Try it**](https://dotnetfiddle.net/SooSeu) |
| `AuditDisplayFormat(string dataFormatString)` | Attribute to change the Audit property display format. | [Try it**](https://dotnetfiddle.net/LT6aSE) |
| `AuditExclude` | Attribute to exclude from the audit the entity or property. | [Try it**](https://dotnetfiddle.net/Dcldvf) |
| `AuditInclude` | Attribute to include in the audit the entity or property. | [Try it**](https://dotnetfiddle.net/Dcldvf) |

### Extension Methods
| Name | Description | Example |
| :--- | :---------- | :------ |
| `Where<TEntityType>(this DbSet<AuditEntry> set)` | Gets the audit trail of all entries entry of entity type `TEntityType`. | [Try it**](https://dotnetfiddle.net/SooSeu) |
| `Where<TEntityType>(this DbSet<AuditEntry> set, TEntityType entry)` | Gets the audit trail of the specific entry. | [Try it**](https://dotnetfiddle.net/LT6aSE) |
| `Where<TEntityType>(this DbSet<AuditEntry> set, params object[] keyValues)` | Gets the audit trail of the specific key. | [Try it**](https://dotnetfiddle.net/Dcldvf) |
| `Where<TEntityType>(this DbSet<XmlAuditEntry> set)` | Gets the audit trail of all entries entry of entity type `TEntityType`. | [Try it**](https://dotnetfiddle.net/SooSeu) |
| `ToLog(this AuditEntry entry)` | Returns a string representation of the audit that can be easily saved in a log file. | [Try it**](https://dotnetfiddle.net/SooSeu) |
| `ToAuditEntries(this IEnumerable<XmlAuditEntry> items)` | Return a list of `XmlAuditEntry` converted into `AuditEntry`. | [Try it**](https://dotnetfiddle.net/SooSeu) |

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
