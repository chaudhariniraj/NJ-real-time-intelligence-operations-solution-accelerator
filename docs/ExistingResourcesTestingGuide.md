# Existing Resources Testing Guide

This guide provides step-by-step test scenarios for QA to validate the "use existing resources" functionality in the Real-Time Intelligence Operations Solution Accelerator.

## Prerequisites

- Azure subscription with Contributor access
- Azure CLI installed and authenticated (`az login`)
- Azure Developer CLI (azd) installed
- Access to Microsoft Fabric

## Test Environment Setup

Before running tests, ensure you have a clean environment:

```bash
# Clone the repository (if not already done)
git clone https://github.com/microsoft/real-time-intelligence-operations-solution-accelerator.git
cd real-time-intelligence-operations-solution-accelerator
```

---

## Test Scenarios

### Scenario 1: Deploy All New Resources (Baseline)

**Purpose:** Establish baseline deployment with all new resources created.

**Steps:**

1. Create a new azd environment:
   ```bash
   azd env new test-scenario1
   ```

2. Deploy without any existing resource variables:
   ```bash
   azd up
   ```

3. Note down the created resources for use in subsequent scenarios:
   ```bash
   azd env get-values
   ```
   
   Record these values:
   - `AZURE_FABRIC_CAPACITY_NAME`: _______________
   - `AZURE_EVENT_HUB_NAMESPACE_NAME`: _______________
   - `AZURE_EVENT_HUB_NAME`: _______________
   - `FABRIC_WORKSPACE_NAME`: _______________

**Expected Results:**
- [ ] New Resource Group created
- [ ] New Fabric Capacity created
- [ ] New Event Hub Namespace created
- [ ] New Event Hub created
- [ ] New Fabric Workspace created
- [ ] All Fabric components deployed (Eventhouse, Dashboard, Eventstream, etc.)
- [ ] Deployment completes successfully

---

### Scenario 2: Use All Existing Resources

**Purpose:** Verify deployment with all existing resources including workspace.

**Prerequisites:** Complete Scenario 1 and note all resource names.

**Steps:**

1. Create a new azd environment:
   ```bash
   azd env new test-scenario2
   ```

2. Set all existing resources:
   ```bash
   azd env set EXISTING_FABRIC_CAPACITY_NAME "<capacity-name-from-scenario1>"
   azd env set EXISTING_EVENT_HUB_NAMESPACE_NAME "<namespace-name-from-scenario1>"
   azd env set EXISTING_EVENT_HUB_NAME "<eventhub-name-from-scenario1>"
   azd env set FABRIC_WORKSPACE_NAME "<workspace-name-from-scenario1>"
   ```

3. Deploy:
   ```bash
   azd up
   ```

**Expected Results:**
- [ ] New Resource Group created
- [ ] **NO** new Fabric Capacity created (uses existing)
- [ ] **NO** new Event Hub Namespace created (uses existing)
- [ ] **NO** new Event Hub created (uses existing)
- [ ] **NO** new Fabric Workspace created (uses existing)
- [ ] Deployment completes (idempotent - components already exist)
- [ ] Output `USING_EXISTING_FABRIC_CAPACITY` is `true`
- [ ] Output `USING_EXISTING_EVENT_HUB_NAMESPACE` is `true`
- [ ] Output `USING_EXISTING_EVENT_HUB` is `true`

**Verification:**
```bash
azd env get-values | grep USING_EXISTING
# Should show all three as true
```

---

### Scenario 3: Existing FC + Existing Event Hub Namespace + Existing Event Hub + New Workspace

**Purpose:** Verify deployment uses all existing Azure resources but creates a new Fabric Workspace.

**Prerequisites:** Complete Scenario 1 and note all resource names.

**Steps:**

1. Create a new azd environment:
   ```bash
   azd env new test-scenario3
   ```

2. Set existing Azure resources (but NOT workspace):
   ```bash
   azd env set EXISTING_FABRIC_CAPACITY_NAME "<capacity-name-from-scenario1>"
   azd env set EXISTING_EVENT_HUB_NAMESPACE_NAME "<namespace-name-from-scenario1>"
   azd env set EXISTING_EVENT_HUB_NAME "<eventhub-name-from-scenario1>"
   # Note: NOT setting FABRIC_WORKSPACE_NAME - will create new workspace
   ```

3. Deploy:
   ```bash
   azd up
   ```

**Expected Results:**
- [ ] New Resource Group created
- [ ] **NO** new Fabric Capacity created (uses existing)
- [ ] **NO** new Event Hub Namespace created (uses existing)
- [ ] **NO** new Event Hub created (uses existing)
- [ ] **NEW** Fabric Workspace created
- [ ] All Fabric components deployed to the new workspace
- [ ] Output `USING_EXISTING_FABRIC_CAPACITY` is `true`
- [ ] Output `USING_EXISTING_EVENT_HUB_NAMESPACE` is `true`
- [ ] Output `USING_EXISTING_EVENT_HUB` is `true`

**Verification:**
```bash
azd env get-values | grep USING_EXISTING
# Should show all three as true

# Verify new workspace was created (different from scenario 1)
azd env get-values | grep FABRIC_WORKSPACE
```

---

### Scenario 4: New FC + Existing Event Hub Namespace + Existing Event Hub + New Workspace

**Purpose:** Verify using existing Event Hub resources while creating new Fabric Capacity and Workspace.

**Prerequisites:** Complete Scenario 1 and note Event Hub resource names.

**Steps:**

1. Create a new azd environment:
   ```bash
   azd env new test-scenario4
   ```

2. Set only Event Hub resources:
   ```bash
   azd env set EXISTING_EVENT_HUB_NAMESPACE_NAME "<namespace-name-from-scenario1>"
   azd env set EXISTING_EVENT_HUB_NAME "<eventhub-name-from-scenario1>"
   # Note: NOT setting EXISTING_FABRIC_CAPACITY_NAME - will create new capacity
   # Note: NOT setting FABRIC_WORKSPACE_NAME - will create new workspace
   ```

3. Deploy:
   ```bash
   azd up
   ```

**Expected Results:**
- [ ] New Resource Group created
- [ ] **NEW** Fabric Capacity created
- [ ] **NO** new Event Hub Namespace created (uses existing)
- [ ] **NO** new Event Hub created (uses existing)
- [ ] **NEW** Fabric Workspace created
- [ ] All Fabric components deployed successfully
- [ ] Output `USING_EXISTING_FABRIC_CAPACITY` is `false`
- [ ] Output `USING_EXISTING_EVENT_HUB_NAMESPACE` is `true`
- [ ] Output `USING_EXISTING_EVENT_HUB` is `true`

**Verification:**
```bash
azd env get-values | grep USING_EXISTING
# Should show:
# USING_EXISTING_FABRIC_CAPACITY=false
# USING_EXISTING_EVENT_HUB_NAMESPACE=true
# USING_EXISTING_EVENT_HUB=true
```

---

### Scenario 5: Existing FC + Existing Event Hub Namespace + New Event Hub + New Workspace

**Purpose:** Verify creating a new Event Hub in an existing Event Hub Namespace.

**Prerequisites:** Complete Scenario 1 and note Fabric Capacity and Event Hub Namespace names.

**Steps:**

1. Create a new azd environment:
   ```bash
   azd env new test-scenario5
   ```

2. Set existing Fabric Capacity and Event Hub Namespace (but NOT Event Hub):
   ```bash
   azd env set EXISTING_FABRIC_CAPACITY_NAME "<capacity-name-from-scenario1>"
   azd env set EXISTING_EVENT_HUB_NAMESPACE_NAME "<namespace-name-from-scenario1>"
   # Note: NOT setting EXISTING_EVENT_HUB_NAME - will create new Event Hub
   # Note: NOT setting FABRIC_WORKSPACE_NAME - will create new workspace
   ```

3. Deploy:
   ```bash
   azd up
   ```

**Expected Results:**
- [ ] New Resource Group created
- [ ] **NO** new Fabric Capacity created (uses existing)
- [ ] **NO** new Event Hub Namespace created (uses existing)
- [ ] **NEW** Event Hub created in the existing namespace
- [ ] **NEW** Fabric Workspace created
- [ ] All Fabric components deployed successfully
- [ ] Output `USING_EXISTING_FABRIC_CAPACITY` is `true`
- [ ] Output `USING_EXISTING_EVENT_HUB_NAMESPACE` is `true`
- [ ] Output `USING_EXISTING_EVENT_HUB` is `false`

**Verification:**
```bash
azd env get-values | grep USING_EXISTING
# Should show:
# USING_EXISTING_FABRIC_CAPACITY=true
# USING_EXISTING_EVENT_HUB_NAMESPACE=true
# USING_EXISTING_EVENT_HUB=false

# Verify new Event Hub was created in existing namespace
az eventhubs eventhub list --namespace-name "<namespace-name-from-scenario1>" --resource-group "<resource-group>" --query "[].name" -o table
# Should show multiple Event Hubs
```

---

## Test Results Summary

| Scenario | Description | Pass/Fail | Notes |
|----------|-------------|-----------|-------|
| 1 | All New Resources | | |
| 2 | All Existing Resources | | |
| 3 | Existing FC + EH + New Workspace | | |
| 4 | New FC + Existing EH + New Workspace | | |
| 5 | Existing FC + Existing EH Namespace + New EH + New Workspace | | |

---

## Cleanup

After testing, clean up resources:

```bash
# Delete each test environment
azd env select test-scenario1
azd down --force --purge

azd env select test-scenario2
azd down --force --purge

azd env select test-scenario3
azd down --force --purge

azd env select test-scenario4
azd down --force --purge

azd env select test-scenario5
azd down --force --purge
```

Or delete resource groups directly:
```bash
az group delete --name <resource-group-name> --yes --no-wait
```

---

## Troubleshooting

### Common Issues

1. **Fabric Capacity not found**
   - Ensure the capacity exists and you have access
   - Verify the name is spelled correctly (case-sensitive)

2. **Event Hub Namespace not in same resource group**
   - Existing Event Hub Namespace must be in the same resource group as the deployment

3. **Workspace already exists with different capacity**
   - If reusing a workspace, ensure it's assigned to the correct capacity

4. **Permission denied**
   - Verify you have Contributor access to the subscription/resource group
   - Verify you have access to the existing resources

### Viewing Deployment Outputs

```bash
# View all environment values
azd env get-values

# Check specific outputs
azd env get-values | grep EXISTING
azd env get-values | grep AZURE_FABRIC
azd env get-values | grep AZURE_EVENT_HUB
```
