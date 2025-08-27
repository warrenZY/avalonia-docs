---
id: bookmarks
title: Bookmarks
---

# Bookmarks

Bookmarks are particularly important in modern operating systems with strict security and privacy controls to maintain access to files and folders. For example, on platforms such as iOS and newer versions of macOS, direct file system access is severely restricted. Instead, an app requests that the user selects a file or folder via a system-provided file picker, and the operating system then gives the app a security-scoped bookmark it can use to access that file or folder in the future.

In Avalonia's StorageProvider, these bookmarks are represented by the IStorageBookmarkFile and IStorageBookmarkFolder interfaces.

The IStorageBookmarkItem Interface
The IStorageBookmarkItem interface represents a bookmarked storage item. It inherits from IStorageItem and IDisposable. This interface is not client implementable, meaning you cannot create your own classes that implement it without special permissions.

Here are the key properties and methods it provides:

Properties:

CanBookmark: A property that indicates if the item can be bookmarked and reused later.

Name: The name of the item.

Path: The file-system path of the item.

How to Use Bookmark Methods
This section provides a practical guide on using the methods of the IStorageBookmarkItem interface, with examples from the provided C# code.

Saving and Loading Bookmarks
To get a bookmark ID for a specific folder or file, use the SaveBookmarkAsync() asynchronous method on a storage item. Once you have a bookmark ID, you can save it to a local database for future use instead of requiring the user to select a folder every time.

SaveBookmarkAsync(): This method is used to get a bookmark ID for a selected file or folder, which can be stored for future use.

private async Task SaveBookmarksAsync(Control control)
{
    // A folder must be selected first
    if (_lastSelectedFolder == null) return;

    var bookmarkId = await _lastSelectedFolder.SaveBookmarkAsync();

    if (bookmarkId != null)
    {
        // Save the bookmarkId to a local file for later use.
        // ... (code to save bookmarkId)
        CurrentActiveBookmarkId = bookmarkId;
    }
}

You can use the OpenFolderBookmarkAsync() methods to open a bookmarked folder via a bookmark ID. This will return the bookmarked file or folder or null if the operating system denies the request.

LoadSelectedBookmarkAsync(): This method is used to open a bookmarked folder and access its contents. It retrieves the stored bookmark ID from your local file and uses it to open the folder.

private async Task LoadSelectedBookmarkAsync(Control control)
{
    if (string.IsNullOrEmpty(SelectedBookmarkId)) return;

    var toplevel = TopLevel.GetTopLevel(control);
    var folder = await toplevel.StorageProvider.OpenFolderBookmarkAsync(SelectedBookmarkId);

    if (folder != null)
    {
        // Successfully opened the bookmarked folder.
        LastSelectedFolder = folder;
        SelectedFolder = folder.Path.LocalPath;
    }
}

ReleaseBookmarkAsync(): This method is used to revoke the security-scoped access granted by the operating system. You should call this when you no longer need access to the bookmarked item.

private async Task ReleaseBookmarkAsync(Control control)
{
    if (!string.IsNullOrEmpty(SelectedBookmarkId))
    {
        // First, remove the ID from local storage.
        // ... (code to remove bookmarkId from file)

        // Then, try to release the OS bookmark.
        var toplevel = TopLevel.GetTopLevel(control);
        if (toplevel?.StorageProvider != null)
        {
            var folder = await toplevel.StorageProvider.OpenFolderBookmarkAsync(SelectedBookmarkId);
            if (folder is IStorageBookmarkItem storageBookmark)
            {
                await storageBookmark.ReleaseBookmarkAsync();
                storageBookmark.Dispose();
            }
        }
    }
}

Managing Bookmarked Files and Folders
Once a bookmark is loaded, you can use the inherited methods from IStorageItem to manipulate the file or folder.

DeleteAsync(): This method asynchronously deletes the current storage item and its contents.

private async Task DeleteFileAsync()
{
    if (SelectedFile != null && LastSelectedFolder != null)
    {
        await SelectedFile.DeleteAsync();
        // Then refresh the UI.
    }
}

MoveAsync(IStorageFolder): This method asynchronously moves the bookmarked item to a new location.

// Example usage (not from provided code, but shows how it works)
// IStorageFile bookmarkedFile = ...;
// IStorageFolder newDestinationFolder = ...;
// await bookmarkedFile.MoveAsync(newDestinationFolder);

GetBasicPropertiesAsync(): This method asynchronously retrieves basic properties of the storage item, such as its size and modification date.

// Example usage
// IStorageFile bookmarkedFile = ...;
// var properties = await bookmarkedFile.GetBasicPropertiesAsync();
// long size = properties.Size;

GetParentAsync(): This method asynchronously gets the parent folder of the current storage item.

// Example usage
// IStorageFile bookmarkedFile = ...;
// var parentFolder = await bookmarkedFile.GetParentAsync();
// string parentName = parentFolder.Name;

TryGetLocalPath(): This extension method attempts to get the local file system path as a string. It's useful for platform-specific operations where a local path is required.

// This will work on Windows but may return null on other platforms
// IStorageFile bookmarkedFile = ...;
// string? localPath = bookmarkedFile.TryGetLocalPath();

Platform-Specific Bookmark Representation
The way a bookmark ID is represented can vary by platform:

Windows: A bookmark is a simple absolute path string.

Android: Think of the content provider like a waiter that your apps can ask for a certain file/folder through a Content URI. The URI format looks like content://[Authority]/[path]/[id]. For example, com.android.externalstorage.documents is an authority for accessing External Storage providers, so a bookmark might look like content://com.android.externalstorage.documents/tree/[...your folder path].
:::note
The exact behavior and capabilities can depend on the specific operating system and its security policies. For instance, on some platforms, a bookmark might become invalid if the user moves or renames the file or folder that it points to.
:::

:::note
It's not recommended to store bookmark IDs in a remote database, as bookmarks might not be persistent and might contain sensitive file path information.
:::
