---
layout: post
title:  "Testing at the edge"
date:   2014-08-01 12:12:12
---

Hints and tips for exploring the corners of your Windows FSD or File System filter.

Several people have recently asked me about testing "into the corners of the
file system", and this article is an attempt to point out some of the darker
corners that you may want to look into while testing file systems and file
system drivers.

Often a change to a popular application or filter will suddenly cause grief on
a hitherto stable filter; sometimes an API will suddenly become popular, and
occasionally you will find someone pushing the edge of your design envelope.
In any case it is always better to head this off in the lab.

So here is my top 10 list of things to think about when testing (and indeed
when designing) an FSD or a file system filter. Not all of them will apply to
you, but some of them will.

### 1. Test at the Native API as well at Win32.

During early development I find it best to test at the native API – that way
you get a one to one correspondence between operations that you send and IRPs
that you see. I cannot speak too highly of Ladislav Zezula’s FileTest utility
(available at OsrOnline and at Ladislav’s own site). This it exactly what you
need to get going. As an added bonus the source is available so you can add
calls for anything your product needs (for instance driver-specific ioctls and
fsctls).

Once you have your driver more or less working you will want to write
application level tests. Whether these tests are written at the Native API
level (NtCreateFile, and so on) or the Win32 API level (CreateFile) will
probably depend on whether the test is being written by a driver developer or
not.

If you write tests to the Win32 API you will be testing a subset of the calls
which your filesystem or filter can see, but you will be testing using the
paradigms used by a majority of the applications.

If you write tests at the Native API level you will be able to test the entire
breadth of the API (and it is broad - just consider, for instance, the
different ways to rename a file). This can be particularly important when
testing capabilities which are not widely available when you test but suddenly
acquire a Win32 API.

When testing against the Native API, you should be aware that the behaviours
are not documented, and may not be what you expect, so always test against a
reference implementation (usually NTFS) as well.

No matter how you test, I urge that you test using both APIs with the balance
between the two being driven by the needs of your specific product.

## 2. Streams, Dispositions and Delete on Close

If your filter or file system handles streams (and most filters need to) you
may need to make yourself aware of the ways in which the presence of a stream
in a file name presented for create processing affects the operation of an
FSD.

I am not going to discuss the precise semantics; they are not documented and
hence may change. My experience is that although the behaviour is usually what
you might expect after some thought about the problems, it is often not
immediately intuitive. This is because the API doesn’t support streams as
first class objects and so the precise semantics have had to be subject to
best-guess approximations.

I recommend that you perform the nature study appropriate to your driver and
then test as seems fit but here are few places to start thinking about.

An open SUPERCEDE to a file is impacted by the file sharing constraints on all
opens to all streams. Similarly, OVERWRITE, OVERWRITE_IF and delete on close
can have wider impact that just the specified file.

It is my experience that a filter just has to adopt a pessimistic view in such
situations (if a bad thing might happen, assume that it will). FSDs which seek
to emulate the streams behaviour of NTFS have no other option than to test
every single possible combination and emulate that.

## 3. IoCancelCreate & Stack based file objects

These are well described elsewhere (see for instance here). In general,
filters should be very careful about using the file objects that they get
given during create.

Don’t do reads and writes using them.

Before you post an operation requiring a file object to another thread or
borrow a file object for any purpose which will outlast the call frame you are
in, ensure that it is not stack based.

You should note that, contrary to what may have been stated elsewhere, it is
still possible to be given stack based file objects in Vista.

## 4. Redirector behaviour

Because the API is the same, it is tempting to assume that filtering a
redirector will be more or less the same as filtering a local file system.
This topic is worthy of an article of its own, but for now here are some items
to ponder:

The same directory or file/stream can have multiple stream contexts.

This is because the redirector cannot necessarily determine that the share is
the same (consider that '\\machine\share\dir\dir2' 'z:\dir\dir2' or
'\\192.1.108.3\share2\dir2' can refer to the same directory). However, even
within one share there can be multiple contexts; the redirector does not know
that a file referenced via a short name is the same as one referenced via a
long name. It may not even be the file name that differs: 'z:\programs
files\file' will have a different stream context to 'z:\progra~1\file'. Hint:
the win16 program 'edit' is a great way to force the use of short names.

Beware of undocumented protocols between various components

You will see this most often in Vista and/or when filtering MUP. These
protocols include (but are not limited to) magic values being passed down in
FsContext and FsContext2 during pre-created and out of band communication
which your filter may never see. Client Side Caching and DFS are two
components which have caused me problems, there are certainly others.

Beware of the limitations of the media

This is the most obvious issue. Redirectors work over unreliable media and so
failures from the FSD should be considered as usual and not exceptional. Run
your filter over RDR, pull out the Ethernet. What happens?

Not all CIFS/SMB servers behave like Windows.

… And even for Windows there have been multiple versions of these subsystems.

## 5. Hard links

Hard links have been around since (at least) NT 3.54. Prior to Vista hard
links tended to be the preserve of the people writing to the native API or
using SFU.

Hard links became much more main-stream in Vista where they acquired a Win32
API (CreateHardLink) and have become a favourite mechanism for the Windows
installer. Fortunately in Vista there are a few more tools to help the filter
driver.

Considering hard links really boils down to two things:

* Don’t assume that the name for a file is unique.
* Remember that delete of a file is really an unlink operation.

In addition, NTFS will give the same Stream Context for a file regardless of
the name used to open it but RDR won’t (as we saw above, a redirector cannot
be aware of all the paths to a file).

In Vista a new QueryInformation (FileHardLinkInformation) call is declared by
ntifs (although no documentation exists). This enumerates all the names for a
file. In using it I have noted a couple of wrinkles you need to be aware of:

Short names are listed as links. This is useful information, but bear in mind
that a short name is a "hybrid" – it is really a second name within a single
hard link. Inexplicably, the FileNameLength in the FILE_LINK_ENTRY_INFORMATION
structure is the number of Unicode character points, not the (more usual) size
in bytes.

## 6. Explicit and implicit volume locking

If your filter keeps handles open to the volume then you are probably already
aware of the need to respond to FSCTL_LOCK_VOLUME (explicit locking). You may
not be aware of the need to respond to an implicit lock – a volume open with
restricted sharing.

The impact of this is best understood by inspecting the example code in the
metadatatool examples shipped with the WDK.

## 7. Case sensitivity

Since Windows 2000, case sensitivity in NTFS has been turned off in the
registry and it has become a easy to ignore this issue.

However, SFU enables and enforces case sensitivity and it seems likely that
NFS (more widely available in Vista) will have similar behaviour. If your file
system or filter manipulates file names, then you should run some case
sensitivity tests. IFSTest has a few case sensitive tests and is a good place
to start.

In general, case sensitivity devolves to propagating the logical not of
FO_OPENED_CASE_SENSITIVE into the RtlXXXUnicodeString functions.

## 8. Files which can require special handling

Depending on their function, filter drivers may need to handle certain files
in particular ways. Often filters will decide that the best solution is to
ignore creates to these files entirely. You should think about special
handling for at least these files:

* NTFS reserved files. This includes the root directory and all files opened below the $Extend pseudo-directory.
* Volume opens. These always require special processing. As noted above, implicit locking is a side effect of specific sorts of volume opens.
* Sparse files. Backup filters in particular need to be aware of sparse files and to take the appropriate actions when dealing with them.
* Encrypted files. Opening files which are encrypted by NTFS involves an undocumented protocol between a worker process and NTFS. Depending on precisely how active your filter is during create processing, you may need to become involved in this protocol if you have to handle encrypted files.
* Compressed files. Whether a file is compressed is largely irrelevant to filters, but it is worth noting that all access to a compressed file is via the cache, even if the file has been opened FO_NO_INTERMEDIATE_BUFFERING.
Reparse points. Again, so long as you correctly handle STATUS_REPARSE, reparse points should be of no great concern to a filter driver. However, if you open files during processing you should be aware of whether or not you need to set the FILE_OPEN_REPARSE_POINT bit.

## 9. Interoperability testing with other filters

Interoperability testing can be open ended. This does not mean that you should
not start it. I would recommend anyone to go to the regular IFS Plugfest
events, which offer an unparalleled opportunity to test with a huge cross
section of other filters. You may find that your first visit flushes several
issues, but you will usually thereafter find that there are a few anti-
patterns you should be avoiding. Of course, attendance at Plugfest is self
selecting and so it is less likely that badly behaved filters (one which hook
for instance) will be present.

And finally…

Don’t forget to run both prefast and pclint over your code, and make sure that
understand what it is saying to you.

If you have any comments on this article, please feel free to send me mail.

Equally, if you need help with planning the test of your driver, or with any
other Windows file system project please feel free to contact Steading System
Software.
