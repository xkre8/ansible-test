📚 Ansible Infrastructure Roles Documentation
🎯 Overview
Collection ของ Ansible roles สำหรับจัดการ infrastructure automation ประกอบด้วย User Management และ LVM Management

🧑‍💻 Role: User Management
📝 Description
Role สำหรับจัดการ user accounts อัตโนมัติ รองรับการสร้าง user, กำหนด groups, และจัดการ home directories

📋 Features
✅ Auto-generate username, UID, password ถ้าไม่ระบุ
✅ ตรวจสอบ groups ที่มีอยู่ในระบบก่อนสร้าง user
✅ รองรับ additional groups
✅ Auto-detect OS และกำหนด shell ที่เหมาะสม
✅ Comprehensive reporting พร้อม track generated values

⚙️ Variables

# Default values
users: []
default_shell: "/bin/bash"
default_group: "users"
default_uid: 1000
uid_min: 1000
uid_max: 9999
password_length: 12
home_base_path: "/home"

# Input variable (YAML string)
user_yml: ""

🔧 Task Breakdown
1. Process Users (process_users.yml)

- name: "Convert user_yml to users list"
  ansible.builtin.set_fact:
    users: "{{ (user_yml | from_yaml) if user_yml is defined and user_yml | length > 5 else users }}"

Condition: ทำงานเมื่อ user_yml มีข้อมูลมากกว่า 5 characters

2. Validate Groups (validate_groups.yml)

- name: "Check groups exist"
  ansible.builtin.command: getent group "{{ item }}"
  register: group_check
  failed_when: false
  when: all_required_groups | length > 0

- name: "Fail if missing groups"
  ansible.builtin.fail:
    msg: "Missing groups: {{ missing_groups }}"
  when: missing_groups | length > 0

Conditions:

ตรวจสอบเมื่อมี groups ที่ต้องการ
หยุดการทำงานถ้ามี groups ที่หายไป
3. Create Users (create_users.yml)

- name: "Process user data"
  vars:
    final_username: "{{ item.username | default('') }}"
    user_data:
      name: "{{ final_username if final_username != '' else random_username }}"
      uid: "{{ final_uid if final_uid != '' else (1000 if final_gid == '' else random_uid) }}"
  when: users | length > 0

Conditions:

ทำงานเมื่อมี users ที่ต้องการสร้าง
ใช้ conditional logic สำหรับ auto-generation
🧪 Test Scenarios
Scenario 1: Complete User Data (PASS)

user_yml: |
  - username: "alice"
    uid: "1001"
    gid: "users"
    password: "mypassword123"
    home_directory: "/home/alice"
    groupname: "wheel"
  - username: "bob"
    gid: "users"
    additional_groups:
      - "wheel"

Scenario 2: Missing Groups (FAIL)
user_yml: |
  - username: "charlie"
    gid: "nonexistent_team"
    additional_groups:
      - "docker"
      - "kubernetes"

💾 Role: LVM Management
📝 Description
Role สำหรับจัดการ Logical Volume Management (LVM) อัตโนมัติ รองรับทั้ง VM ใหม่และ VM เก่า

📋 Features
✅ Auto-detect disk สำหรับ VM ใหม่
✅ เลือกใช้ VG ที่มีอยู่สำหรับ VM เก่า
✅ สร้าง PV, VG, LV อัตโนมัติ
✅ Auto-generate LV names จาก path
✅ รองรับหลาย filesystem types


# Strategy configuration
vm_mode: "new_vm"                    # new_vm, existing_vm
vg_strategy: "create_new"
pv_strategy: "create_new_pv"
volume_group_name: "vg_data"

# Device selection
selected_disk_name: ""
auto_detect_disk: true
existing_vg_name: ""
disk_selection_mode: "index"        # index, name
selected_disk_index: 1

# Filesystem configuration
filesystem_config: |
  - path: /data
    lv_name: lv_data
    size_mb: 5000
    fstype: xfs

🔧 Task Breakdown
1. Device Discovery

- name: "Auto-detect available disks"
  ansible.builtin.shell: |
    lsblk -dpno NAME,SIZE,TYPE | grep disk | grep -v sda | grep -v sdb
  register: available_disks
  when: 
    - vm_mode == "new_vm"
    - auto_detect_disk | default(true)

Condition: ทำงานเมื่อเป็น VM ใหม่และเปิด auto-detect

2. Validation

- name: "Validate device exists"
  ansible.builtin.stat:
    path: "{{ target_device }}"
  register: device_check
  failed_when: not device_check.stat.exists
  when: target_device is defined

Condition: ตรวจสอบเมื่อมีการกำหนด target device

3. LVM Creation

- name: "Create Physical Volume"
  ansible.builtin.command: pvcreate {{ target_device }}
  when: 
    - pv_strategy == "create_new_pv"
    - target_device is defined

- name: "Create Volume Group"
  community.general.lvg:
    vg: "{{ volume_group_name }}"
    pvs: "{{ target_device }}"
    state: present
  when: vg_strategy == "create_new"

Conditions:

สร้าง PV เมื่อ strategy เป็น create_new_pv
สร้าง VG เมื่อ strategy เป็น create_new
🧪 Test Scenarios
Scenario 1: New VM with Auto-detect (PASS)

vm_mode: "new_vm"
volume_group_name: "jbdmvg"
auto_detect_disk: true

filesystem_config: |
  - path: /opt/app
    lv_name: ""                    # จะ auto-generate เป็น jbdmvg_opt_app
    size_mb: 2048
    fstype: xfs
  - path: /data
    lv_name: "custom_data"
    size_mb: 5120
    fstype: ext4

Scenario 2: Missing Disk (FAIL)

vm_mode: "new_vm"
volume_group_name: "testmvg"
auto_detect_disk: false
selected_disk_name: "sdz"          # Disk ที่ไม่มีอยู่
filesystem_config: |
  - path: /test
    size_mb: 1024
    fstype: xfs

/////// Playbook ///////////

User Management Playbook

---
- name: Manage Users
  hosts: all
  become: yes
  tasks:
    - name: Setup User Management
      include_role:
        name: company.infrastructure.user_management
      vars:
        user_yml: |
          - username: "admin"
            gid: "wheel"
            password: "secure123"
          - username: ""              # Will generate random
            gid: ""                   # Will use 'users' + UID 1000
            password: ""              # Will generate random

LVM Management Playbook

---
- name: Setup LVM
  hosts: all
  become: yes
  tasks:
    - name: Configure LVM
      include_role:
        name: company.infrastructure.lvm_management
      vars:
        vm_mode: "new_vm"
        volume_group_name: "app_vg"
        filesystem_config: |
          - path: /opt/application
            size_mb: 10240
            fstype: xfs
          - path: /var/logs
            size_mb: 5120
            fstype: ext4