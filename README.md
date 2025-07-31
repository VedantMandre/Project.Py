```
Summary of Missing Foreign Keys by Table
🔷 group_users (group_id, user_id)
🧩 Likely composite PK: (group_id, user_id)

❌ Missing FKs

Column	References	Type
group_id	groups(group_id)	Individual FK
user_id	users(user_id)	Individual FK

➡️ Must define two individual FKs (different parent tables).

🔷 group_services (group_id, service_id)
🧩 Likely composite PK: (group_id, service_id)

❌ Missing FKs

Column	References	Type
group_id	groups(group_id)	Individual FK
service_id	service_products(service_id)	Individual FK (assuming it exists)

🔷 user_services_view (user_id, branch_id, service_id)
🧩 Appears to be a view-like table (not a true relation)

❌ No FKs

Column	References	Type
user_id	users(user_id)	Individual FK
branch_id	likely branch_products.branch_id or some branches table	Individual FK
service_id	service_products(service_id)	Individual FK

➡️ Requires 3 individual FKs

🔷 user_preferences (user_id, type)
🧩 Likely composite PK: (user_id, type)

❌ Missing FKs

Column	References	Type
user_id	users(user_id)	Individual FK

➡️ type is likely enum-style; user_id should have FK

🔷 branch_products (branch_id, product_id)
🧩 Composite PK

❌ Missing FKs

Column	References	Type
branch_id	(missing branches table?)	Individual FK
product_id	products(product_id)	Individual FK

🔷 company_products (branch_id, sun_id, product_id)
🧩 Composite PK

❌ Missing FKs

Column	References	Type
branch_id	branch_products.branch_id (or separate branches)	Individual FK
sun_id	Used in multiple tables; no clear parent seen	Needs further tracing
product_id	products(product_id)	Individual FK

🔷 company_configurations (sun_id, branch_id, product_id)
🧩 Composite key candidate

❌ Missing FKs

Column	References	Type
sun_id	Possibly from user_accounts, unclear	Needs analysis
branch_id	As above	Individual FK
product_id	products(product_id)	Individual FK

🔷 user_accounts (user_id, sun_id, branch_id)
🧩 Probably not a composite PK, but all likely FKs

❌ Missing FKs

Column	References	Type
user_id	users(user_id)	Individual FK
sun_id	Ambiguous	Needs further tracing
branch_id	As above	Individual FK

🔷 user_configurations (user_id, branch_id)
🧩 Composite PK likely

❌ Missing FKs

Column	References	Type
user_id	users(user_id)	Individual FK
branch_id	Likely from branches	Individual FK
```
