---
title: Binary File Uploads in gRPC-Gateway
impact: HIGH
impactDescription: Critical for enabling standard browser-based file uploads which are not natively handled by gRPC bytes fields.
tags: grpc-gateway, binary-uploads, golang, multipart-form
---

## Binary File Uploads in gRPC-Gateway

**Impact: HIGH**

gRPC handles binary data as `bytes` fields, but standard `multipart/form-data` uploads (e.g., from a web browser or `curl -F`) cannot be directly mapped to a gRPC method call. You must bypass standard mapping and handle these manually.

**Correct (Manual Handler Registration):**

Register a manual path on the `runtime.ServeMux` instance and implement a handler that parses the multipart form.

```go
// Registration in Gateway Setup
mux := runtime.NewServeMux()
err := mux.HandlePath("POST", "/v1/files/upload", handleBinaryFileUpload)

// Handler Implementation
func handleBinaryFileUpload(w http.ResponseWriter, r *http.Request, params map[string]string) {
    if err := r.ParseForm(); err != nil {
        http.Error(w, "invalid form", http.StatusBadRequest)
        return
    }
    file, header, err := r.FormFile("attachment")
    if err != nil {
        http.Error(w, "missing attachment", http.StatusBadRequest)
        return
    }
    defer file.Close()
    // Process the file stream...
}
```

**Correct (OpenAPI Documentation via Dummy RPC):**

Since manual routes aren't automatically documented, use a "dummy" RPC to generate the OpenAPI specification for the upload endpoint.

```protobuf
rpc UploadFile(google.protobuf.Empty) returns (google.protobuf.Empty) {
  option (google.api.http) = {
    post: "/v1/files/upload"
  };
  option (grpc.gateway.protoc_gen_openapiv2.options.openapiv2_operation) = {
    summary: "Upload a binary file"
    consumes: "multipart/form-data"
    parameters: {
      key: "attachment"
      value: {
        name: "attachment"
        in: "formData"
        required: true
        type: FILE
      }
    }
  };
}
```

_Note: The actual implementation in the gateway must still use the manual handler approach even if the dummy RPC is defined._

Reference: [Binary File Uploads](https://grpc-ecosystem.github.io/grpc-gateway/docs/mapping/binary_file_uploads/)
