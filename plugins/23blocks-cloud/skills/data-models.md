---
name: data-models
description: Design and implement 23blocks data models and migrations
---

# 23blocks Data Models

Design database schemas following 23blocks conventions.

## Model Conventions

### Standard Fields
Every model should include:
```ruby
create_table :resources, id: :uuid do |t|
  t.string :name, null: false
  t.references :organization, type: :uuid, foreign_key: true
  t.references :created_by, type: :uuid, foreign_key: { to_table: :users }

  t.datetime :deleted_at  # Soft delete
  t.timestamps            # created_at, updated_at
end

add_index :resources, :deleted_at
add_index :resources, [:organization_id, :name], unique: true
```

### Naming Conventions
- Tables: plural, snake_case (`user_profiles`)
- Columns: snake_case (`first_name`)
- Foreign keys: `{table}_id` (`organization_id`)
- Indexes: auto-named or `index_{table}_on_{columns}`

## Relationships

### belongs_to
```ruby
class Resource < ApplicationRecord
  belongs_to :organization
  belongs_to :created_by, class_name: 'User'
end
```

### has_many
```ruby
class Organization < ApplicationRecord
  has_many :resources, dependent: :destroy
  has_many :users, through: :memberships
end
```

### Polymorphic
```ruby
class Comment < ApplicationRecord
  belongs_to :commentable, polymorphic: true
end
```

## Migration Patterns

### Adding columns
```ruby
add_column :users, :phone, :string
add_index :users, :phone
```

### Safe migrations
```ruby
# Use disable_ddl_transaction for concurrent indexes
disable_ddl_transaction!

def change
  add_index :resources, :external_id,
            algorithm: :concurrently,
            if_not_exists: true
end
```

## Usage

When designing data models:
1. Identify entities and relationships
2. Apply 23blocks conventions
3. Add appropriate indexes
4. Consider soft deletes
5. Plan for multi-tenancy (organization_id)
