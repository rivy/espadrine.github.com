# TheFileTree Design Log 3: Collaborative Editing

I finally implemented the following API:

- `GET /file?op=edit&app=text` (WebSocket): use the [Canop][] protocol to synchronize the file's content and autosave it.

[Canop]: https://github.com/espadrine/canop

To get there, I improved Canop to finalize its protocol and survive disconnections. I made it more resilient to edge-cases and streamlined its API. I even added the ability to create brand new fundamental operations as a Canop user.

I chose to go with [Canop][] instead of [jsonsync][], even though I appeared to lean towards the latter in previous design logs, because implementing index rebasing is non-trivial (especially for jsonsync), and I had already mostly done so with Canop.

On the other hand, it only supports strings. The protocol is designed to allow full JSON data, however. Additionally, strings are by far the harders data type to implement synchronization for. Arrays are slightly easier (and the index rebasing necessary to make them work is the same as that for strings), and the others are trivial.

[jsonsync]: https://github.com/espadrine/jsonsync

## User Interface

I have designed Canop with certain events to allow for the display of the synchronization state (unsyncable (ie, offline), syncing, and synced).

This was problematic in thefiletree.com previously, as it was not at all clear whether we had suddenly been disconnected, and there were no reconnection. Closing the tab could therefore cause us to lose our changes… Yet there was no save button.

## Details

I don't save the edit history server-side, so local operations after a disconnection won't be able to rebase on top of other operations. I get the feeling that past a few seconds of disconnection, concurrent editing can severely damage the user's intended edition, as rebased operation may lose what the user entered without them having seen someone's cursor arrive.

(Of course, nothing precludes someone from deleting the whole file, removing your changes, but that is malice, not accident. What we want to forbid are accidental loss. Malicious changes are inevitable; or rather, being resistant to them requires complex graph synchronization, solving the byzantine generals problem, and saving the full non-linear edit history.)

I don't even save the file's revision number, so the client cannot tell whether there has been zero changes since the disconnection and apply local changes from there.

My plan: save the revision number so that local changes can be applied without a sweat when the file wasn't concurrently modified.

If it was, download a diffing library, and perform a diff. Display a visualization of the changes that would be applied to the local document (with local changes) if upstream changes were applied, and local changes rebased on top of them.

"Do you want to apply remote changes?" Yes / No, save my document at a new location.

<script type="application/ld+json">
{ "@context": "http://schema.org",
  "@type": "BlogPosting",
  "datePublished": "2017-04-02T19:19:00Z",
  "keywords": "tree" }
</script>
