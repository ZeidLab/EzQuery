# EzQuery
## What is EzQuery ?
 > __EzQuery__  is a  super easy to use  yet powerful .Net Core 5 library,
 to build SQL queries, to execute them , and to return populated sets of results of given
 table models.

## Supported data annotation attributes
>- Table
>- Key
>- Column
>- NotMapped
>- Required
## What is a table model?
>In order to use this library you need to define a table model.
>Below is a model example of **Users** Table containing **Name,LastName,BirthDay,Credit** and a not mapped properties named **Status** :

```csharp
    [Table("Users")]
    public class User :EzQueryTable<User>
    {
        [Key]
        public int Id { get; set; }
        [Column("Name")]
        [Required]
        public string FirstName { get; set; }
        public string? LastName { get; set; }
        public DateTime? BirthDay { get; set; }
        public decimal Credit { get; set; }
        [NotMapped]
        public bool Status { get; set; }
    }
```