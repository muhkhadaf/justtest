-- ============================================================
-- RENAME SCHEMA: invenlist.XXXX -> dbo.invenlist_XXXX
-- Jalankan di SSMS pada database yang bersangkutan
-- ============================================================

-- LANGKAH 1: Pindahkan dari schema 'invenlist' ke schema 'dbo'
-- LANGKAH 2: Rename nama tabelnya dengan prefix 'invenlist_'

-- Jika schema 'dbo' belum ada (biasanya sudah ada by default):
-- CREATE SCHEMA dbo;

-- -------------------------------------------------------
-- Tabel: invenlist.drafts -> dbo.invenlist_drafts
-- -------------------------------------------------------
ALTER SCHEMA dbo TRANSFER invenlist.drafts;
EXEC sp_rename 'dbo.drafts', 'invenlist_drafts';

-- -------------------------------------------------------
-- Tabel: invenlist.log_data -> dbo.invenlist_log_data
-- -------------------------------------------------------
ALTER SCHEMA dbo TRANSFER invenlist.log_data;
EXEC sp_rename 'dbo.log_data', 'invenlist_log_data';

-- -------------------------------------------------------
-- Tabel: invenlist.device_hp_monthly_recap -> dbo.invenlist_device_hp_monthly_recap
-- -------------------------------------------------------
ALTER SCHEMA dbo TRANSFER invenlist.device_hp_monthly_recap;
EXEC sp_rename 'dbo.device_hp_monthly_recap', 'invenlist_device_hp_monthly_recap';

-- -------------------------------------------------------
-- Tabel: invenlist.imei_history -> dbo.invenlist_imei_history
-- -------------------------------------------------------
ALTER SCHEMA dbo TRANSFER invenlist.imei_history;
EXEC sp_rename 'dbo.imei_history', 'invenlist_imei_history';

-- -------------------------------------------------------
-- Tabel: invenlist.ctd_phone_types -> dbo.invenlist_ctd_phone_types
-- -------------------------------------------------------
ALTER SCHEMA dbo TRANSFER invenlist.ctd_phone_types;
EXEC sp_rename 'dbo.ctd_phone_types', 'invenlist_ctd_phone_types';

-- -------------------------------------------------------
-- VERIFIKASI: Cek semua tabel setelah rename
-- -------------------------------------------------------
SELECT TABLE_SCHEMA, TABLE_NAME 
FROM INFORMATION_SCHEMA.TABLES 
WHERE TABLE_TYPE = 'BASE TABLE'
ORDER BY TABLE_SCHEMA, TABLE_NAME;
