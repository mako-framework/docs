# Upgrading

--------------------------------------------------------

* [Uploaded files](#uploaded_files)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `6.2.x` to `6.3.x`.

--------------------------------------------------------

<a id="uploaded_files"></a>

### Uploaded files

The `UploadedFile::getName()` and `UploadedFile::getReportedType()` are deprecated and will be removed in Mako `7.0`. You can update your application and make it future-proof by replacing the method calls with `UploadedFile::getReportedFilename()` and `UploadedFile::getReportedMimeType()`.