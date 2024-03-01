# Blazor file streaming

Sample inspired by [documentation for Work with images in ASP.NET Core Blazor](https://learn.microsoft.com/en-us/aspnet/core/blazor/images) extended to work with any file type that browser is able to preview.

## The code

From Home.razor
```csharp
private async Task<(Stream, string?)> DownloadFileAsync(string url)
{
    var absoluteUrl = NavigationManager.ToAbsoluteUri(url);
    Console.WriteLine($"Downloading file from {absoluteUrl}");

    var response = await Http.GetAsync(absoluteUrl);
    string? contentType = null;
    if (response.Content.Headers.TryGetValues("Content-Type", out var values))
        contentType = values.FirstOrDefault();

    return (await response.Content.ReadAsStreamAsync(), contentType);
}

private async Task ShowFileAsync(string url)
{
    var (imageStream, contentType) = await DownloadFileAsync(url);
    
    var dotnetImageStream = new DotNetStreamReference(imageStream);
    await JS.InvokeVoidAsync("setIframeSource", "iframe", dotnetImageStream, contentType);
}
```

From index.html
```javascript
window.setIframeSource = async (imageElementId, imageStream, contentType) => {
  const arrayBuffer = await imageStream.arrayBuffer();
  let blobOptions = {};
  if (contentType) {
    blobOptions['type'] = contentType;
  }
  const blob = new Blob([arrayBuffer], blobOptions);
  const url = URL.createObjectURL(blob);
  const image = document.getElementById(imageElementId);
  image.onload = () => {
    URL.revokeObjectURL(url);
  }
  image.src = url;
}
```

## Live

Running at https://maraf.github.io/blazor-file-stream/