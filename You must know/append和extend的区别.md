# Python中append和extend的区别

## list.append(object) 向列表中添加一个对象object

## list.extend(sequence) 把一个序列seq的内容添加到列表中

```
music_media = ['compact disc', '8-track tape', 'long playing record']
new_media = ['DVD Audio disc', 'Super Audio CD']
music_media.append(new_media)
print music_media
>>>['compact disc', '8-track tape', 'long playing record', ['DVD Audio disc', 'Super Audio CD']]
```

使用append的时候，是将new_media看作一个对象，整体打包添加到music_media对象中。



```	
music_media = ['compact disc', '8-track tape', 'long playing record']
new_media = ['DVD Audio disc', 'Super Audio CD']
music_media.extend(new_media)
print music_media
>>>['compact disc', '8-track tape', 'long playing record', 'DVD Audio disc', 'Super Audio CD']
```

使用extend的时候，是将new_media看作一个序列，将这个序列和music_media序列合并，并放在其后面。