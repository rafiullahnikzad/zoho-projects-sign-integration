# Zoho Projects to Zoho Sign Integration

Automate document signing workflows by fetching task attachments from Zoho Projects and sending them to clients via Zoho Sign for e-signature.

## Overview

This Zoho Deluge script automates the document signing process by:

1. **Fetching** PDF attachments from a Zoho Projects task
2. **Downloading** files from Zoho WorkDrive
3. **Uploading** all PDFs to Zoho Sign as a single signing request
4. **Configuring** signature fields with precise placement
5. **Sending** the document package to the client for signature

## Use Case

Ideal for businesses that need to:
- Send project deliverables for client approval
- Collect signatures on project documentation
- Automate the handoff from project completion to client sign-off

## Prerequisites

### Zoho Connections Required

| Connection Name | Service | Scope |
|-----------------|---------|-------|
| `projects_conn` | Zoho Projects | `ZohoProjects.portals.ALL`, `ZohoProjects.tasks.ALL` |
| `workdriveconn` | Zoho WorkDrive | `ZohoWorkDrive.files.READ` |
| `zohosign` | Zoho Sign | `ZohoSign.documents.ALL`, `ZohoSign.templates.ALL` |

### Setup Connections

1. Go to **Zoho Creator** → **Settings** → **Developer Space** → **Connections**
2. Create connections for each service with appropriate scopes
3. Authorize each connection with your Zoho account

## Configuration

Update these variables in the script:

```javascript
portalId = "your-portal-name";           // Zoho Projects portal ID
projectId = "300973000000XXXXXX";         // Project ID
taskId = "300973000000XXXXXX";            // Task ID containing attachments
clientEmail = "client@example.com";       // Recipient's email
clientName = "Client Name";               // Recipient's display name
```

### Finding Your IDs

| ID | How to Find |
|----|-------------|
| **Portal ID** | URL: `projects.zoho.eu/portal/{portalId}/...` |
| **Project ID** | URL: `...#projectdetail/{projectId}/...` or via API |
| **Task ID** | URL: `...#taskdetail/{projectId}-{taskId}/...` or via API |

## Workflow Diagram

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Zoho Projects  │────▶│  Zoho WorkDrive │────▶│   Zoho Sign     │
│  (Get Task      │     │  (Download      │     │  (Create &      │
│   Attachments)  │     │   PDF Files)    │     │   Send Request) │
└─────────────────┘     └─────────────────┘     └─────────────────┘
         │                      │                       │
         ▼                      ▼                       ▼
   List attached          Download each           Upload all PDFs
   files metadata         PDF content             as single request
                                                        │
                                                        ▼
                                                  Add signature
                                                  field to last
                                                  page, send to
                                                  client
```

## Code Walkthrough

### Step 1: Get Task Attachments

```javascript
attachUrl = "https://projectsapi.zoho.eu/api/v3/portal/" + portalId + 
            "/projects/" + projectId + "/tasks/" + taskId + "/attachments";

attachResponse = invokeurl
[
    url: attachUrl
    type: GET
    connection: "projects_conn"
];

attachmentsList = attachResponse.get("attachment");
```

Retrieves metadata for all files attached to the specified task.

### Step 2: Download PDFs from WorkDrive

```javascript
for each attachment in attachmentsList
{
    fileName = attachment.get("name");
    fileType = attachment.get("type");
    workdriveFileId = attachment.get("third_party_file_id");
    
    if(fileType.contains("pdf"))
    {
        wdDownloadUrl = "https://workdrive.zoho.eu/api/v1/download/" + workdriveFileId;
        
        headerMap = Map();
        headerMap.put("Accept", "application/octet-stream");
        
        fileContent = invokeurl
        [
            url: wdDownloadUrl
            type: GET
            headers: headerMap
            connection: "workdriveconn"
        ];
        
        pdfFilesList.add(fileContent);
        pdfNames.add(fileName);
    }
}
```

Downloads each PDF file using the WorkDrive file ID stored in the attachment metadata.

### Step 3: Upload to Zoho Sign

```javascript
signResponse = zoho.sign.createDocument(pdfFilesList, Map(), "zohosign");

requestId = signResponse.get("requests").get("request_id");
documentIds = signResponse.get("requests").get("document_ids");
```

Uploads all PDFs in a single request, creating a multi-document signing package.

### Step 4: Configure Signature Field

```javascript
fieldObj = Map();
fieldObj.put("field_name", "Signature");
fieldObj.put("field_type_name", "Signature");
fieldObj.put("field_type_id", signFieldId);
fieldObj.put("document_id", lastDocId);
fieldObj.put("page_no", lastPageNo);
fieldObj.put("is_mandatory", true);
fieldObj.put("x_coord", 295);
fieldObj.put("y_coord", 595);
fieldObj.put("abs_width", 350);
fieldObj.put("abs_height", 55);
```

Places the signature field on the last page of the last document. Adjust coordinates based on your document layout.

### Step 5: Submit for Signing

```javascript
actionObj = Map();
actionObj.put("recipient_name", clientName);
actionObj.put("recipient_email", clientEmail);
actionObj.put("action_type", "SIGN");
actionObj.put("fields", fieldList);

// Submit the request
submitUrl = "https://sign.zoho.eu/api/v1/requests/" + requestId + "/submit";
```

Assigns the signer and sends the document for signature.

## Signature Field Positioning

The signature coordinates are calibrated for documents with a "Unterschrift:" (Signature) label.

| Parameter | Value | Description |
|-----------|-------|-------------|
| `x_coord` | 295 | Horizontal position from left edge |
| `y_coord` | 595 | Vertical position from top |
| `abs_width` | 350 | Signature field width |
| `abs_height` | 55 | Signature field height |

### Adjusting Position

To customize signature placement:

1. Create a test document in Zoho Sign manually
2. Position the signature field where desired
3. Use Zoho Sign API to retrieve the field coordinates
4. Update the coordinates in the script

## API Endpoints Used

| Service | Endpoint | Purpose |
|---------|----------|---------|
| Projects | `/api/v3/portal/{portal}/projects/{project}/tasks/{task}/attachments` | Get task attachments |
| WorkDrive | `/api/v1/download/{fileId}` | Download file content |
| Sign | `zoho.sign.createDocument()` | Upload documents |
| Sign | `zoho.sign.getFieldIds()` | Get field type IDs |
| Sign | `/api/v1/requests/{requestId}/submit` | Submit for signing |

## Regional Domains

Update URLs based on your Zoho data center:

| Region | Projects API | WorkDrive API | Sign API |
|--------|-------------|---------------|----------|
| EU | `projectsapi.zoho.eu` | `workdrive.zoho.eu` | `sign.zoho.eu` |
| US | `projectsapi.zoho.com` | `workdrive.zoho.com` | `sign.zoho.com` |
| IN | `projectsapi.zoho.in` | `workdrive.zoho.in` | `sign.zoho.in` |
| AU | `projectsapi.zoho.com.au` | `workdrive.zoho.com.au` | `sign.zoho.com.au` |

## Response Examples

### Successful Upload Response

```json
{
  "status": "success",
  "requests": {
    "request_id": "300973000000XXXXXX",
    "document_ids": [
      {
        "document_id": "300973000000XXXXXX",
        "total_pages": 5
      }
    ]
  }
}
```

### Successful Submit Response

```json
{
  "code": 0,
  "message": "Request submitted successfully",
  "requests": {
    "request_status": "inprogress"
  }
}
```

## Error Handling

The script includes basic error checking:

```javascript
if(signResponse.get("status") == "success")
{
    // Process successful upload
}
else
{
    info "❌ Upload failed: " + signResponse;
}

if(submitResp.get("code") == 0)
{
    info "✅ SUCCESS!";
}
else
{
    info "❌ Submit failed: " + submitResp;
}
```

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Connection error | Invalid/expired OAuth | Re-authorize the connection |
| No attachments found | Wrong task ID | Verify task ID in URL |
| Download fails | WorkDrive permissions | Check file sharing settings |
| Invalid field position | Coordinates out of bounds | Adjust x_coord/y_coord |

## Customization Options

### Multiple Signers

```javascript
// Add multiple recipients
actionList = List();

action1 = Map();
action1.put("recipient_name", "First Signer");
action1.put("recipient_email", "first@example.com");
action1.put("action_type", "SIGN");
action1.put("signing_order", 1);
actionList.add(action1);

action2 = Map();
action2.put("recipient_name", "Second Signer");
action2.put("recipient_email", "second@example.com");
action2.put("action_type", "SIGN");
action2.put("signing_order", 2);
actionList.add(action2);
```

### Additional Field Types

```javascript
// Date field
dateField = Map();
dateField.put("field_type_name", "Date");
dateField.put("field_name", "Date Signed");
// ... other properties

// Text field
textField = Map();
textField.put("field_type_name", "Textfield");
textField.put("field_name", "Full Name");
// ... other properties
```

## Deployment

### As a Standalone Function

1. Create a new function in Zoho Creator or CRM
2. Paste the script
3. Configure connections
4. Execute manually or via workflow

### As a Workflow Trigger

1. Create a workflow rule on task status change
2. Add a Deluge function action
3. Call this function when task status = "Completed"

### As a Scheduled Function

1. Go to **Settings** → **Automation** → **Schedules**
2. Create a new schedule
3. Select this function
4. Set frequency (daily, weekly, etc.)

## License

MIT License - Feel free to use and modify for your projects.

## Author

**Rafiullah Nikzad**  
Senior Zoho Developer | CloudZ Technologies

- Portfolio: [rafiullahnikzad.netlify.app](https://rafiullahnikzad.netlify.app)
- LinkedIn: [Zoho Afghanistan Community](https://www.linkedin.com/company/zoho-afghanistan)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Support

For questions or issues:
1. Open an issue in this repository
2. Contact via LinkedIn
3. Refer to [Zoho Deluge Documentation](https://www.zoho.com/deluge/help/)
