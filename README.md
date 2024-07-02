- 👋 Hi, I’m @Envera112
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
Envera112/Envera112 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->Hi , I'm @Envera112
I'm interested in creating a multi user sql database that has limited and restricted user public access 

-- SQL Code to create a table for MG Mali Attorney files with specified criteria

-- Assuming the existence of a users table, where 'is_superuser' flag denotes admin with full access
-- and 'can_create_trust_files' denotes permission to create trust files.

-- Create a table for trust files if it doesn't exist
CREATE TABLE IF NOT EXISTS trust_files (
    file_id INT AUTO_INCREMENT PRIMARY KEY,
    file_name VARCHAR(255) NOT NULL,
    created_by_admin_id INT NOT NULL,
    date_created DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (created_by_admin_id) REFERENCES users(user_id)
    -- Ensure that the user has permissions to create trust files
    CHECK ((SELECT is_superuser FROM users WHERE users.user_id = created_by_admin_id) = 1)
);

-- Insert trigger to check if the user creating the trust file is an administrator with superuser privilege.
DELIMITER $$
CREATE TRIGGER before_trust_file_creation
BEFORE INSERT ON trust_files
FOR EACH ROW
BEGIN
    DECLARE v_superuser BOOLEAN;
    SELECT is_superuser INTO v_superuser FROM users WHERE user_id = NEW.created_by_admin_id;
    IF NOT v_superuser THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Only administrators with superuser privileges can create trust files.';
    END IF;
END$$
DELIMITER ;

-- Grant limited access to admins for capturing Source specialists and uploading documents.
-- Assume document_uploads is a separate table for uploading documents related to trust files.
CREATE TABLE IF NOT EXISTS document_uploads (
    upload_id INT AUTO_INCREMENT PRIMARY KEY,
    file_id INT NOT NULL,
    document BLOB NOT NULL,
    uploaded_by_admin_id INT NOT NULL,
    date_uploaded DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (file_id) REFERENCES trust_files(file_id),
    FOREIGN KEY (uploaded_by_admin_id) REFERENCES users(user_id)
    -- Only allow certain admins to upload documents
    -- Specific business logic to determine which admins can upload would be required here
);

-- Adjust the necessary privileges for the admin users to capture the source specialists.
-- Assuming source_specialists is another table for this purpose.
CREATE TABLE IF NOT EXISTS source_specialists (
    specialist_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    captured_by_admin_id INT NOT NULL,
    associated_file_id INT NOT NULL,
    date_captured DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (associated_file_id) REFERENCES trust_files(file_id),
    FOREIGN KEY (captured_by_admin_id) REFERENCES users(user_id)
    -- Business rules to limit which admins can capture source specialists go here.
);
