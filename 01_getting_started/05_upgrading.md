# Upgrading

--------------------------------------------------------

* [Redirects](#redirects)
* [Uploaded files](#uploaded_files)
* [Validation rules](#validation_rules)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `6.2.x` to `6.3.x`. The release does not contain any breaking changes but there are a couple of deprecations. Follow this upgrade guide to future-proof your application.

--------------------------------------------------------

<a id="redirects"></a>

### Redirects

The `Redirect::multipleChoices()`, `Redirect::notModified()` and `Redirect::useProxy()` methods have been deprecated and will be removed in Mako `7.0`.

--------------------------------------------------------

<a id="uploaded_files"></a>

### Uploaded files

The `UploadedFile::getName()` and `UploadedFile::getReportedType()` are deprecated and will be removed in Mako `7.0`. You can replace the method calls with `UploadedFile::getReportedFilename()` and `UploadedFile::getReportedMimeType()`.

--------------------------------------------------------

<a id="validation_rules"></a>

### Validation rules

The `max_filesize` and `mimetype` rules have been deprecated and will be removed in Mako `7.0`. They have been renamed to `max_file_size` and `mime_type`.