# Ansible for CMDB and System Reporting

A 100% Ansible-native CMDB reporting tool that generates static HTML (and JSON) system reports using Ansible facts. No external tools or databases required — just Ansible and Jinja2 templates.

## How It Works

1. Ansible gathers facts from target hosts during playbook execution
2. Jinja2 templates render per-host HTML reports and a central index page
3. Reports are delegated and written directly to the `[report]` server
4. The report server (RHEL + Nginx) serves the static reports over HTTP

## Output Formats

- **HTML** — Per-host reports + index page (available now)
- **JSON** — Per-host JSON + index manifest for machine-readable consumption (planned)
- **CSV** — Tabular export (planned)
- **SQL** — Database-compatible export (planned)

## Requirements

- Ansible (via Execution Environment container)
- `ansible-navigator` for local execution
- Ansible Automation Platform (AAP) for scheduled/production execution

## Usage

### Using ansible-navigator (Recommended)

```bash
# Run against all nodes
# consider forks value and capacity of the Ansible node / Execution node
ansible-navigator run generate-cmdb-report.yml -e "target_nodes=all"

# Run against a host group
ansible-navigator run generate-cmdb-report.yml -e "target_nodes=rhel8" -m stdout

# Run with a custom execution environment
ansible-navigator run generate-cmdb-report.yml -e "target_nodes=rhel8" --eei <custom-ee-image> -m stdout

# Override report output path
ansible-navigator run generate-cmdb-report.yml -e "target_nodes=rhel8" -e "system_report_path=/var/www/cmdb-reports" -m stdout
```

### Using Ansible Automation Platform (AAP)

1. Add this repository as a **Project** in AAP
2. Create a **Job Template** pointing to `generate-cmdb-report.yml`
3. Set extra variables: `target_nodes: <target_group>`
4. Optionally schedule the job template for periodic report generation

### Report Server

Reports are delegated directly to the first host in the `[report]` inventory group. Configure a RHEL host with Nginx serving the report output directory (default: `/var/www/cmdb-reports`).

**Inventory setup:**

```ini
[report]
reportserver01 ansible_host=192.168.57.3 ansible_user=devops
```

**Quick start with Docker/Podman (for local testing):**

```bash
docker-compose up -d
# Reports available at http://localhost:8089
```

## Project Structure

```
├── generate-cmdb-report.yml    # Main playbook
├── hosts                       # Inventory file
├── templates/
│   ├── system-report.html      # Per-host report template
│   ├── system-report-index.html # Index page template
│   └── html/                   # Jinja2 template fragments
│       ├── 00-head.html        #   CSS and page head
│       ├── 01-header.html      #   Per-host page header
│       ├── 01-header-home.html #   Index page header
│       ├── system_information.html # System facts table
│       ├── index-content.html  #   Index listing
│       └── 99-footer.html      #   Page footer
├── web/                        # Sample generated reports
├── docker-compose.yaml         # Nginx container for serving reports
├── ansible.cfg                 # Ansible configuration
├── execution-environment.yml   # Custom EE definition
├── requirements.yml            # Collection dependencies
└── requirements.txt            # Python dependencies (for EE)
```

## Variables

| Variable | Default | Description |
|---|---|---|
| `target_nodes` | `no-such-hosts` | Target host group to report on |
| `system_report_path` | `/var/www/cmdb-reports` | Output directory on the report server |

## License

See [LICENSE](LICENSE) for details.
