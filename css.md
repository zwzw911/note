##### firefox tab  
实现类似firxfox tab的效果：当选中tab时，此tab的top/left/right有边框，bottom无边框。暗示用户此tab和下面的内容关联。  
其他tab，当hover时，显示所有边框，提示用户此tab可选，但是尚未选择（因为bottom的border是有线的）。  
**解决方案：**  
难点在于内容和tab重合的部分，设置为无边框（而不是内容的top为无边框）。  
利用tab来遮盖内容的上边框来实现内容的部分边框不可见：  
1. tab的bottom无边框。  
2. tab的margin-bottom设为负值，以便让内容框向上，和tab重合的部分处于tab之下。
3. tab必须设置background-color，这样，重合部分才会被遮盖；没有设置的话，默认是transparent，内容的边框还是会显示。
