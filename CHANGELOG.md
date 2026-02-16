# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-02-17

### Added
- Initial release
- Fetch task attachments from Zoho Projects API v3
- Download PDF files from Zoho WorkDrive
- Upload multiple PDFs to Zoho Sign as single signing request
- Auto-detect signature field type ID
- Configurable signature field placement
- Single signer workflow with email notification
- Comprehensive error handling and logging
- EU region API endpoints support

### Technical Details
- Uses `invokeurl` for Projects and WorkDrive API calls
- Uses `zoho.sign.createDocument()` integration task for Sign upload
- Uses `zoho.sign.getFieldIds()` for field type retrieval
- Direct API call for Sign request submission

## [Unreleased]

### Planned Features
- Multiple signers support
- Additional field types (date, text, initials)
- US/IN/AU region support
- Workflow trigger integration
- Email template customization
- Callback/webhook for signature completion
