fuselage
========

tag-based semantic file system, in python, over FUSE. Pluggable aspects (parsing or interpretting). Querying and relations between tags/items. Modular architecture, allows different user-ends (fs/webdav/http/..), or backends (sql/rdf/..)

2011, hristo stanev + svilen dobrev


Fuselage (tagging file system)
==============================

## Installation

* Install the dependencies:
  sudo apt-get install python-fuse fuse-utils python-kjbuckets
* Get the source
 * download, then extract (into directory ./fs/)
    tar xvf fuselage.tar.gz
 * or from bzr repository (use tagfs/fs/mount.py instead of fs/mount.py) 
    bzr co http+webdav://94.190.190.193/bzr/tagfs/trunk/ tagfs
* Create a directory for the mount point
  mkdir ~/myfs
* Mount it (repository is ~/.fsrepo by default)
  fs/mount.py ~/myfs
** for another repository:
   ... -o repository=<path_to_repository> ...
** to turn on logging:
   ... -o logfile=<path_to_logfile> ...


== Available aspects/views/directories

In the mount point we have directories each representing a single
aspect. These can be enabled/disabled from a list in config.py. Some
aspects work with related objects and so they allow for filtering
on relations to reach certain object. I call such aspects QAspects.
Queries in QAspects are defined by type of operands and type of
results. Logical intersection, union and negation can
be applied over operands to scope the set of results. All QAspects
share the same syntax for queries and the grammar is customizable in
config.py.

* archive - works with items (files or directories)
* tags -  A QAspect with tags as operands and items as results. Use it to create/remove/rename tags as well as to query items.
* mintags - same as tags but only showing those intersecting with the path so far
* hardlink - modifies other aspects by creating hardlinks instead of symlinks
* items - A QAspect similar to Tags, but providing the opposite view over the same relation. Items are presented as directories and act as operands to find tags which are the result type.
* untagged - A simple view which gives all items that have no tags. I am not sure if this deserves to be a separate aspect, but may be useful.
* parentOf - childrenOf - sameAs - QAspects dealing with the hierarchy of tags. ParentOf uses tags as operands and returns tags that are considered super tags to those in the query tree. ChildrenOf returns subtags and SameAs returns aliases or equal tags.
* metatags - A QAspect which allows for tagging tags themself.Here operands are called metatags and results are tags. The result of such tagging is a special kind of compound tags which carry a view over the tag being used. With such compound tags you can say "this image was made at Bart's birthday" which doesn't mean Bart was on the image, but uses the tag Bart" as place. Here "place" is the metatag applied over "Bart". Currently this is achieved by autocreating compound tags in Tags aspect for every metatagged tag e.g. if Bart was tagged as Place the result would be a new compound tag named Place_Bart in Tags aspect.
* queries - Here the user can save long and complex queries and later call them by just addressing their short names, so he doesn't need to invent them again.


## Basic operations (replace leading ./ with actual mount-point path)

* create tag t
   _mkdir ./tags/t_

* del tag t
   _rmdir ./tags/t_

* create tags t1,t2  
   _mkdir -p ./tags/t1/t2_

* del tags t1,t2 - optional todo: delete t1 and t2; - dangerous
    _rmdir ./tags/t1/t2_ - currently deletes t2 only

* import external items
    (ln /ext/item internal...     WILL NOT WORK - across filesystems)
    (mv /ext/item internal...     WILL NOT WORK - we store only metadata, not the files themselves
    (... so use ln -s into /hardlink/)
    _ln -s /ext/item ./hardlink/archive/_       : becomes hardlink
    _ln -s /ext/item ./hardlink/tags/t1_        : becomes hardlink tagged with t1
    _ln -s /ext/item ./archive/_                : becomes symlink
    _ln -s /ext/item ./tags/t1_                 : becomes symlink tagged with t1
    _cp -rs /ext/item/dirs ./hardlink/tags/_    : dirs become tags, (flat) items become links to ext-tree tagged accordingly
    _cp -rs /ext/item/dirs ./hardlink/archive/_ : dirs => dirs, (tree) items =? symlinks to ext-tree.. is this usable/needed??

* del item (and untag)
    _rm ./archive/item_

* add tags to items
    (ext items: see import above)
    tag item with t2, t3 and t4:   
    _ln .../path/to/item ./tags/t2/t3/t4_ 
    _ln -s .../path/to/item ./tags/t2/t3/t4_  

* del tags x1,x2 from item and add tags t1 and t2 to it
    _mv ./tags/x1/x2/=/item ./tags/t1/t2_ 

* del tags t1,t2 from item   
    _rm ./tag/t1/t2/=/item_
    _rm ./tag/t1/+/t2/=/item_

* del all tags from item - TODO
    _rm ./item2tags/item/*_
    _mv .../query/to/item ./untagall_     ??

* rename item
    _mv ./archive/item ./archive/item2_
    _mv .../query/to/item .../same/query/to/item2_
    _mv ./tags/t1/t2/=/item ./archive/item2_  : will also del tags t1 and t2 from item

* rename tag
    _mv ./tag/t1 ./tag/t3_

* re-tag items, e.g. a/b/c/* to become /abc/ or ab/* to become a/b/
    _mkdir ./tags/abc_
    _mv ./tags/a/+/b/+/c/=/* ./tags/abc_
    _rmdir ./tags/a ./tags/b ./tags/c_

    or, some tags-relation-operation - todo


* make B a subtag of A so that tag A would also yield files tagged as B:
mv tags/A parentOf/B

* make 'Rock' equal to RockNRoll
mv tags/Rock sameAs/RockNRoll

* query relations 
ls sameAs/Rock/=
ls childrenOf/RockNRoll/+/Rock/=
ls parentsOf/RockNRoll/=

* create metatag
mkdir metatags/Person

* Setup tag A as Person
mv tags/A metatags/Person/

* list 'Person' tags
ls metatags/Person/=

* list 'Person' and 'Place' or not 'Brand' tags
ls metatags/Person/Place/+/!/Brand/=

* tag a file f1 as Person_A
ln archive/f1 tags/Person_A

* create a saved query
ln -s tags/A/+/B/+/!C/= queries/Q1

* use a saved query
ls queries/Q1

* list queries
ls queries


## usual usecase

* mass import (cp -rs /ext* ./hardlink/<archive|tags>/)
* re-tagging (a->b, a/b/->ab, ab->a/b/)
* browse (./mintags/)

