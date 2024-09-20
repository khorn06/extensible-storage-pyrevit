# extensible-storage-pyrevit
Utilities for managing extensible storage in Autodesk Revit, designed to work with pyRevit

## Example Usage

### script.py

```python
import System
from pyrevit import DB, revit
from extensible_storage import Entity

from schemas import MySchema, EntitySchema

test_data = {
    "int32": 123,
    "int16": 12345,
    "byte": 255,
    "double": 123.456,
    "single": 123.45,
    "boolean": True,
    "string": "Test String",
    "guid": System.Guid.NewGuid(),
    "element_id": DB.ElementId(123456),
    "xyz": DB.XYZ(1.0, 2.0, 3.0),
    "uv": DB.UV(4.0, 5.0),
    "entity": Entity(DB.ExtensibleStorage.Entity(EntitySchema.schema)),
}

document = revit.doc
with revit.Transaction("Create data storage", document):
    data_storage = DB.ExtensibleStorage.DataStorage.Create(document)

# Open a context on an Entity to ensure a transaction is opened
# to save the entity to the element
with MySchema(data_storage) as entity:
    for value_type, value in test_data.items():
        field_name = "{}_field".format(value_type.lower())

        # Set the value in the entity this way if you need control of the units
        entity.set(field_name, value, unit_type_id=DB.UnitTypeId.General)

    entity.simple_field_example = 123
    entity.array_field_example = [1, 2, 3]
    entity.map_field_example = {"key1": 1, "key2": 2, "key3": 3}

# Get the data on the entity
entity = MySchema(data_storage)
for value_type in test_data:
    field_name = "{}_field".format(value_type.lower())

    # Get the value in the entity this way if you need control of the units
    field_value = entity.get(field_name, unit_type_id=DB.UnitTypeId.General)
    print("{}: {} {}".format(field_name, field_value, type(field_value)))

array_field_value = entity.array_field_example
print("Array Field: {}".format(array_field_value))
map_field_value = entity.map_field_example
print("Map Field: {}".format(map_field_value))
```

### schemas.py

```python
import System
from pyrevit import DB
from extensible_storage import ES, BaseSchema, array_field, map_field, simple_field


class EntitySchema(BaseSchema):
    """Schema for entities"""

    guid = "6fe14499-0028-4e24-9587-e08ba6edaa9a"

    @simple_field(value_type=int)
    def value():
        """An example of a simple field with ES.Entity"""


class MySchema(BaseSchema):
    """This class documentation is used to for the Schema Documentation"""

    guid = System.Guid("a395b68b-6e6a-48bf-9b73-cc89f9937e78")

    read_access_level = ES.AccessLevel.Public
    write_access_level = ES.AccessLevel.Public

    @simple_field(value_type="int32")
    def int32_field():
        """An example of a simple field with System.Int32"""

    @simple_field(value_type="int16")
    def int16_field():
        """An example of a simple field with System.Int16"""

    @simple_field(value_type="byte")
    def byte_field():
        """An example of a simple field with System.Byte"""

    @simple_field(value_type="double", spec_type_id=DB.SpecTypeId.Number)
    def double_field():
        """An example of a simple field with System.Double"""

    @simple_field(value_type="single", spec_type_id=DB.SpecTypeId.Number)
    def single_field():
        """An example of a simple field with System.Single"""

    @simple_field(value_type="boolean")
    def boolean_field():
        """An example of a simple field with System.Boolean"""

    @simple_field(value_type="string")
    def string_field():
        """An example of a simple field with System.String"""

    @simple_field(value_type="guid")
    def guid_field():
        """An example of a simple field with System.Guid"""

    @simple_field(value_type="element_id")
    def element_id_field():
        """An example of a simple field with DB.ElementId"""

    @simple_field(value_type="xyz", spec_type_id=DB.SpecTypeId.Number)
    def xyz_field():
        """An example of a simple field with DB.XYZ"""

    @simple_field(value_type="uv", spec_type_id=DB.SpecTypeId.Number)
    def uv_field():
        """An example of a simple field with DB.UV"""

    @simple_field(value_type="entity", guid=EntitySchema.guid)
    def entity_field():
        """An example of a simple field with DB.ExtensibleStorage.Entity"""

    @array_field(value_type=int)
    def array_field_example():
        """An example of an array field"""

    @map_field(key_type=str, value_type=int)
    def map_field_example():
        """An example of a map field"""
