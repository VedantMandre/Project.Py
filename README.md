```
-- group_users
ALTER TABLE administration.group_users
ADD CONSTRAINT fk_group_users_group_id
FOREIGN KEY (group_id) REFERENCES administration.groups(group_id);

ALTER TABLE administration.group_users
ADD CONSTRAINT fk_group_users_user_id
FOREIGN KEY (user_id) REFERENCES administration.users(user_id);

-- group_services
ALTER TABLE administration.group_services
ADD CONSTRAINT fk_group_services_group_id
FOREIGN KEY (group_id) REFERENCES administration.groups(group_id);

ALTER TABLE administration.group_services
ADD CONSTRAINT fk_group_services_service_id
FOREIGN KEY (service_id) REFERENCES administration.service_products(service_id);

-- user_session
ALTER TABLE administration.user_session
ADD CONSTRAINT fk_user_session_user_id
FOREIGN KEY (user_id) REFERENCES administration.users(user_id);

-- branch_products
ALTER TABLE administration.branch_products
ADD CONSTRAINT fk_branch_products_product_id
FOREIGN KEY (product_id) REFERENCES administration.products(product_id);

-- company_products
ALTER TABLE administration.company_products
ADD CONSTRAINT fk_company_products_product_id
FOREIGN KEY (product_id) REFERENCES administration.products(product_id);

-- company_configurations
ALTER TABLE administration.company_configurations
ADD CONSTRAINT fk_company_configurations_product_id
FOREIGN KEY (product_id) REFERENCES administration.products(product_id);

-- user_accounts
ALTER TABLE administration.user_accounts
ADD CONSTRAINT fk_user_accounts_user_id
FOREIGN KEY (user_id) REFERENCES administration.users(user_id);

-- user_configurations
ALTER TABLE administration.user_configurations
ADD CONSTRAINT fk_user_configurations_user_id
FOREIGN KEY (user_id) REFERENCES administration.users(user_id);

```
