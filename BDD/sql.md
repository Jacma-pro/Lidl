-- WARNING: This schema is for context only and is not meant to be run.
-- Table order and constraints may not be valid for execution.

CREATE TABLE public.audit_log (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  actor_id bigint,
  actor_role text,
  action character varying,
  target_table character varying,
  target_id bigint,
  ip_address text,
  created_at timestamp with time zone,
  CONSTRAINT audit_log_pkey PRIMARY KEY (id)
);
CREATE TABLE public.backup_audit_log (
  id bigint,
  actor_id bigint,
  actor_role text,
  action character varying,
  target_table character varying,
  target_id bigint,
  ip_address text,
  created_at timestamp with time zone
);
CREATE TABLE public.backup_cart (
  id bigint,
  client_id bigint,
  store_id bigint,
  status text,
  created_at timestamp with time zone,
  updated_at timestamp with time zone
);
CREATE TABLE public.backup_cartitem (
  id bigint,
  cart_id bigint,
  product_id bigint,
  quantity bigint,
  unit_price double precision
);
CREATE TABLE public.backup_category (
  id bigint,
  name text,
  restrictions text,
  description text
);
CREATE TABLE public.backup_client (
  id bigint,
  permission_id bigint,
  last_name text,
  first_name text,
  email text,
  password text,
  phone bigint,
  address text,
  preferred_store_id bigint,
  created_at timestamp with time zone,
  updated_at timestamp with time zone
);
CREATE TABLE public.backup_client_account (
  id bigint,
  client_id bigint,
  is_verified boolean,
  is_active boolean,
  created_at timestamp with time zone,
  last_login_at timestamp with time zone,
  updated_at timestamp with time zone
);
CREATE TABLE public.backup_client_history (
  id bigint,
  client_id bigint,
  loyalty_points bigint,
  loyalty_status boolean,
  created_at timestamp with time zone,
  updated_at timestamp with time zone
);
CREATE TABLE public.backup_manager (
  id bigint,
  permission_id bigint,
  store_id bigint,
  first_name text,
  first_name_initials text,
  work_email text,
  password text,
  work_phone bigint,
  created_at timestamp with time zone,
  updated_at timestamp with time zone
);
CREATE TABLE public.backup_notification (
  id bigint,
  client_id bigint,
  order_id bigint,
  type text,
  channel text,
  status text,
  sent_at timestamp with time zone,
  created_at timestamp with time zone
);
CREATE TABLE public.backup_order (
  id bigint,
  client_id bigint,
  store_id bigint,
  pickup_slot_id bigint,
  preparer_id bigint,
  status text,
  total_price double precision,
  pickup_code character varying,
  created_at timestamp with time zone,
  updated_at timestamp with time zone
);
CREATE TABLE public.backup_orderitem (
  id bigint,
  order_id bigint,
  product_id bigint,
  quantity bigint,
  unite_price double precision,
  substitution_id bigint
);
CREATE TABLE public.backup_payment (
  id bigint,
  order_id bigint,
  amount double precision,
  method text,
  status text,
  transaction_ref text,
  created_at timestamp with time zone
);
CREATE TABLE public.backup_performance (
  id bigint,
  preparer_id bigint,
  orders_prepared_count bigint,
  avg_preparation_time double precision,
  period_start_date date,
  period_end_date date,
  created_at timestamp with time zone,
  error_rate double precision,
  stock_shortage_reported bigint,
  global_score double precision
);
CREATE TABLE public.backup_permission (
  id bigint,
  role text,
  created_at timestamp with time zone,
  updated_at timestamp with time zone
);
CREATE TABLE public.backup_pickupslot (
  id bigint,
  store_id bigint,
  date date,
  start_time time without time zone,
  end_time time without time zone,
  max_orders bigint,
  current_orders bigint,
  is_available boolean
);
CREATE TABLE public.backup_product (
  id bigint,
  category_id bigint,
  name text,
  description text,
  price double precision,
  weight double precision,
  lenght double precision,
  width double precision,
  height double precision,
  picture_url bigint,
  barcode bigint,
  nutriscore text,
  is_active boolean,
  created_at timestamp with time zone,
  updated_at timestamp with time zone
);
CREATE TABLE public.backup_product_backup (
  id bigint,
  category_id bigint,
  name text,
  description text,
  price double precision,
  weight double precision,
  lenght double precision,
  width double precision,
  height double precision,
  picture_url bigint,
  barcode bigint,
  nutriscore text,
  is_active boolean,
  created_at timestamp with time zone,
  updated_at timestamp with time zone
);
CREATE TABLE public.backup_schedule (
  id bigint,
  preparer_id bigint,
  store_id bigint,
  date date,
  start_time time without time zone,
  end_time time without time zone,
  status text,
  comment character varying,
  created_at timestamp with time zone,
  updated_at timestamp with time zone
);
CREATE TABLE public.backup_stock (
  id bigint,
  store_id bigint,
  product_id bigint,
  available_quantity bigint,
  updated_at timestamp with time zone
);
CREATE TABLE public.backup_store (
  id bigint,
  name text,
  email text,
  phone bigint,
  address text,
  zip_code bigint,
  city text,
  country text,
  latitude double precision,
  longitude double precision,
  opening_hours text,
  slot_duration_minutes bigint,
  max_orders_per_slot bigint,
  avg_preparation_time_minutes bigint,
  drive_available boolean,
  click_collect_available boolean,
  created_at timestamp with time zone,
  updated_at timestamp with time zone
);
CREATE TABLE public.backup_substitutionproposal (
  id bigint,
  order_item_id bigint,
  original_product_id bigint,
  proposed_product_id bigint,
  status text,
  created_at timestamp with time zone
);
CREATE TABLE public.cart (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  client_id bigint,
  store_id bigint,
  status text,
  created_at timestamp with time zone,
  updated_at timestamp with time zone,
  CONSTRAINT cart_pkey PRIMARY KEY (id),
  CONSTRAINT cart_client_id_fkey FOREIGN KEY (client_id) REFERENCES public.client(id),
  CONSTRAINT cart_store_id_fkey FOREIGN KEY (store_id) REFERENCES public.store(id)
);
CREATE TABLE public.cart_item (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  cart_id bigint,
  product_id bigint,
  quantity bigint,
  unit_price double precision,
  CONSTRAINT cart_item_pkey PRIMARY KEY (id),
  CONSTRAINT cartitem_cart_id_fkey FOREIGN KEY (cart_id) REFERENCES public.cart(id),
  CONSTRAINT cartitem_product_id_fkey FOREIGN KEY (product_id) REFERENCES public.product(id)
);
CREATE TABLE public.category (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  name text,
  restrictions text,
  description text,
  CONSTRAINT category_pkey PRIMARY KEY (id)
);
CREATE TABLE public.client (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  permission_id bigint,
  last_name text,
  first_name text,
  email text,
  password text,
  phone bigint,
  address text,
  preferred_store_id bigint,
  created_at timestamp with time zone,
  updated_at timestamp with time zone,
  CONSTRAINT client_pkey PRIMARY KEY (id),
  CONSTRAINT client_permission_id_fkey FOREIGN KEY (permission_id) REFERENCES public.permission(id)
);
CREATE TABLE public.client_account (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  client_id bigint,
  is_verified boolean,
  is_active boolean,
  created_at timestamp with time zone,
  last_login_at timestamp with time zone,
  updated_at timestamp with time zone,
  CONSTRAINT client_account_pkey PRIMARY KEY (id),
  CONSTRAINT client_account_client_id_fkey FOREIGN KEY (client_id) REFERENCES public.client(id)
);
CREATE TABLE public.client_history (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  client_id bigint,
  loyalty_points bigint,
  loyalty_status boolean,
  created_at timestamp with time zone,
  updated_at timestamp with time zone,
  CONSTRAINT client_history_pkey PRIMARY KEY (id),
  CONSTRAINT client_history_client_id_fkey FOREIGN KEY (client_id) REFERENCES public.client(id)
);
CREATE TABLE public.manager (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  permission_id bigint,
  store_id bigint,
  first_name text,
  last_name_initials text,
  work_email text,
  password text,
  work_phone bigint,
  created_at timestamp with time zone,
  updated_at timestamp with time zone,
  CONSTRAINT manager_pkey PRIMARY KEY (id),
  CONSTRAINT manager_permission_id_fkey FOREIGN KEY (permission_id) REFERENCES public.permission(id),
  CONSTRAINT manager_store_id_fkey FOREIGN KEY (store_id) REFERENCES public.store(id)
);
CREATE TABLE public.notification (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  client_id bigint,
  order_id bigint,
  type text,
  channel text,
  status text,
  sent_at timestamp with time zone,
  created_at timestamp with time zone,
  CONSTRAINT notification_pkey PRIMARY KEY (id),
  CONSTRAINT notification_client_id_fkey FOREIGN KEY (client_id) REFERENCES public.client(id),
  CONSTRAINT notification_order_id_fkey FOREIGN KEY (order_id) REFERENCES public.order(id)
);
CREATE TABLE public.order (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  client_id bigint,
  store_id bigint,
  pickup_slot_id bigint,
  preparer_id bigint,
  status text,
  total_price double precision,
  pickup_code character varying,
  created_at timestamp with time zone,
  updated_at timestamp with time zone,
  CONSTRAINT order_pkey PRIMARY KEY (id),
  CONSTRAINT order_client_id_fkey FOREIGN KEY (client_id) REFERENCES public.client(id),
  CONSTRAINT order_store_id_fkey FOREIGN KEY (store_id) REFERENCES public.store(id),
  CONSTRAINT order_pickup_slot_id_fkey FOREIGN KEY (pickup_slot_id) REFERENCES public.pickup_slot(id),
  CONSTRAINT order_preparer_id_fkey FOREIGN KEY (preparer_id) REFERENCES public.preparer(id)
);
CREATE TABLE public.order_item (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  order_id bigint,
  product_id bigint,
  quantity bigint,
  unit_price double precision,
  substitution_id bigint,
  CONSTRAINT order_item_pkey PRIMARY KEY (id),
  CONSTRAINT orderitem_order_id_fkey FOREIGN KEY (order_id) REFERENCES public.order(id),
  CONSTRAINT orderitem_product_id_fkey FOREIGN KEY (product_id) REFERENCES public.product(id)
);
CREATE TABLE public.payment (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  order_id bigint,
  amount double precision,
  method text,
  status text,
  transaction_ref text,
  created_at timestamp with time zone,
  CONSTRAINT payment_pkey PRIMARY KEY (id),
  CONSTRAINT payment_order_id_fkey FOREIGN KEY (order_id) REFERENCES public.order(id)
);
CREATE TABLE public.performance (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  preparer_id bigint,
  orders_prepared_count bigint,
  avg_preparation_time double precision,
  period_start_date date,
  period_end_date date,
  created_at timestamp with time zone,
  error_rate double precision,
  stock_shortages_reported bigint,
  global_score double precision,
  CONSTRAINT performance_pkey PRIMARY KEY (id),
  CONSTRAINT performance_preparer_id_fkey FOREIGN KEY (preparer_id) REFERENCES public.preparer(id)
);
CREATE TABLE public.permission (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  role text,
  created_at timestamp with time zone,
  updated_at timestamp with time zone,
  CONSTRAINT permission_pkey PRIMARY KEY (id)
);
CREATE TABLE public.pickup_slot (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  store_id bigint,
  date date,
  start_time time without time zone,
  end_time time without time zone,
  max_orders bigint,
  current_orders bigint,
  is_available boolean,
  CONSTRAINT pickup_slot_pkey PRIMARY KEY (id),
  CONSTRAINT pickupslot_store_id_fkey FOREIGN KEY (store_id) REFERENCES public.store(id)
);
CREATE TABLE public.pickup_slots (
  id integer NOT NULL DEFAULT nextval('pickup_slots_id_seq'::regclass),
  store_id integer NOT NULL,
  date date NOT NULL,
  start_time time without time zone NOT NULL,
  end_time time without time zone NOT NULL,
  max_orders integer NOT NULL,
  current_orders integer NOT NULL DEFAULT 0,
  is_available boolean NOT NULL DEFAULT true,
  CONSTRAINT pickup_slots_pkey PRIMARY KEY (id)
);
CREATE TABLE public.preparer (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  permission_id bigint,
  store_id bigint,
  first_name text,
  last_name_initials text,
  work_email text,
  password text,
  work_phone bigint,
  created_at timestamp with time zone,
  updated_at timestamp with time zone,
  CONSTRAINT preparer_pkey PRIMARY KEY (id),
  CONSTRAINT preparer_store_id_fkey FOREIGN KEY (store_id) REFERENCES public.store(id),
  CONSTRAINT preparer_permission_id_fkey FOREIGN KEY (permission_id) REFERENCES public.permission(id)
);
CREATE TABLE public.product (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  category_id bigint,
  name text,
  description text,
  price double precision,
  weight double precision,
  lenght double precision,
  width double precision,
  height double precision,
  image_url character varying,
  barcode bigint,
  nutriscore text,
  is_active boolean,
  created_at timestamp with time zone,
  updated_at timestamp with time zone,
  CONSTRAINT product_pkey PRIMARY KEY (id),
  CONSTRAINT products_category_id_fkey FOREIGN KEY (category_id) REFERENCES public.category(id)
);
CREATE TABLE public.product_backup (
  id bigint,
  category_id bigint,
  name text,
  description text,
  price double precision,
  weight double precision,
  lenght double precision,
  width double precision,
  height double precision,
  picture_url bigint,
  barcode bigint,
  nutriscore text,
  is_active boolean,
  created_at timestamp with time zone,
  updated_at timestamp with time zone
);
CREATE TABLE public.schedule (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  preparer_id bigint,
  store_id bigint,
  date date,
  start_time time without time zone,
  end_time time without time zone,
  status text,
  comment character varying,
  created_at timestamp with time zone,
  updated_at timestamp with time zone,
  CONSTRAINT schedule_pkey PRIMARY KEY (id),
  CONSTRAINT schedule_preparer_id_fkey FOREIGN KEY (preparer_id) REFERENCES public.preparer(id),
  CONSTRAINT schedule_store_id_fkey FOREIGN KEY (store_id) REFERENCES public.store(id)
);
CREATE TABLE public.stock (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  store_id bigint,
  product_id bigint,
  available_quantity bigint,
  updated_at timestamp with time zone,
  CONSTRAINT stock_pkey PRIMARY KEY (id)
);
CREATE TABLE public.store (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  name text,
  email text,
  phone bigint,
  address text,
  zip_code bigint,
  city text,
  country text,
  latitude double precision,
  longitude double precision,
  opening_hours text,
  slot_duration_minutes bigint,
  max_orders_per_slot bigint,
  avg_preparation_time_minutes bigint,
  drive_available boolean,
  click_collect_available boolean,
  created_at timestamp with time zone,
  updated_at timestamp with time zone,
  CONSTRAINT store_pkey PRIMARY KEY (id)
);
CREATE TABLE public.substitution_proposal (
  id bigint GENERATED ALWAYS AS IDENTITY NOT NULL,
  order_item_id bigint,
  original_product_id bigint,
  proposed_product_id bigint,
  status text,
  created_at timestamp with time zone,
  CONSTRAINT substitution_proposal_pkey PRIMARY KEY (id),
  CONSTRAINT substitutionproposal_order_item_id_fkey FOREIGN KEY (order_item_id) REFERENCES public.order_item(id),
  CONSTRAINT substitutionproposal_original_product_id_fkey FOREIGN KEY (original_product_id) REFERENCES public.product(id),
  CONSTRAINT substitutionproposal_proposed_product_id_fkey FOREIGN KEY (proposed_product_id) REFERENCES public.product(id)
);