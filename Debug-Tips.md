## Useful Chrome Browser applications

* [JSONViewer](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&sqi=2&ved=0CB4QFjAA&url=https%3A%2F%2Fchrome.google.com%2Fwebstore%2Fdetail%2Fjsonview%2Fchklaanhfefbnpoihckbnefhakgolnmc%3Fhl%3Den&ei=okwSVNvNHoWuyATU8IDYCQ&usg=AFQjCNH3ET5JyRh_aKGH_G5Ws5MXENK5bA&sig2=pWaOq0PM1ptzGV5Mln3sZg&bvm=bv.75097201,d.aWw)
* [Github Wiki Search](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=2&sqi=2&ved=0CCcQFjAB&url=https%3A%2F%2Fchrome.google.com%2Fwebstore%2Fdetail%2Fgithub-wiki-search%2Fgdifdhnjmjaidbajhapmbcbnoocoeooc%3Fhl%3Den&ei=zkwSVLqcHo2k8AXotIKIDA&usg=AFQjCNFUY7r_nIUR5aamJ5dLvSMOEHqWMQ&sig2=47EAGQ_pAiJFTsxqq6MOlg&bvm=bv.75097201,d.aWw)
* [POSTMAN REST client](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&sqi=2&ved=0CB4QFjAA&url=https%3A%2F%2Fchrome.google.com%2Fwebstore%2Fdetail%2Fpostman-rest-client%2Ffdmmgilgnpjigdojojpjoooidkmcomcm%3Fhl%3Den&ei=8UwSVO3HAsmkyASY-4GoDg&usg=AFQjCNHaecLwAKk91gpdCY_y1x_ViIrHwQ&sig2=cFmqDhGUuPP_DTYV7-OErg&bvm=bv.75097201,d.aWw)
* [Vim in TextArea with ctrl+enter - write more wiki!](https://chrome.google.com/webstore/detail/wasavi/dgogifpkoilgiofhhhodbodcfgomelhe)

## Text-only tools when GUI is not accessible
* [contrail-logs](Using-contrail-logs-to-debug-Contrail) to gather details when UI access is not possible
* To view xml output with indentation
````
    curl -sq http://localhost:8085/Snh_ItfReq? | python -c 'import sys;import xml.dom.minidom;s=sys.stdin.read();print xml.dom.minidom.parseString(s).toprettyxml()' | less
````
* To locate particular stanza based on child value 
    + (for e.g. interface stanza based on name of vhost0, port uuid or vm uuid)
````
    (export KEY=name VAL=vhost0; curl -s -q http://localhost:8085/Snh_ItfReq? | python -c 'import sys, os;from lxml import etree as et;s=sys.stdin.read();root=et.fromstring(s); print et.tostring(root.xpath(".//%s[text()=\"%s\"]/.." %(os.environ["KEY"], os.environ["VAL"]))[0], pretty_print=True)')
    (export KEY=uuid VAL=29ec90f1-c637-4cc8-8edb-125ce3dcc0f9; curl -s -q http://localhost:8085/Snh_ItfReq? | python -c 'import sys, os;from lxml import etree as et;s=sys.stdin.read();root=et.fromstring(s); print et.tostring(root.xpath(".//%s[text()=\"%s\"]/.." %(os.environ["KEY"], os.environ["VAL"]))[0], pretty_print=True)')
    (export KEY=vm_uuid VAL=a607ec69-95e9-4f8a-9d29-f6632e66fd76; curl -s -q http://localhost:8085/Snh_ItfReq? | python -c 'import sys, os;from lxml import etree as et;s=sys.stdin.read();root=et.fromstring(s); print et.tostring(root.xpath(".//%s[text()=\"%s\"]/.." %(os.environ["KEY"], os.environ["VAL"]))[0], pretty_print=True)')
````

* Vim function to pretty print large XML. Put following in your .vimrc and type \x.
````
" XML formatter
function! DoFormatXML() range
        " Save the file type
        let l:origft = &ft 

        " Clean the file type
        set ft= 

        " Add fake initial tag (so we can process multiple top-level elements)
        exe ":let l:beforeFirstLine=" . a:firstline . "-1"
        if l:beforeFirstLine < 0 
                let l:beforeFirstLine=0
        endif
        exe a:lastline . "put ='</PrettyXML>'"
        exe l:beforeFirstLine . "put ='<PrettyXML>'"
        exe ":let l:newLastLine=" . a:lastline . "+2"
        if l:newLastLine > line('$')
                let l:newLastLine=line('$')
        endif

        " Remove XML header
        exe ":" . a:firstline . "," . a:lastline . "s/<\?xml\\_.*\?>\\_s*//e"

        " Recalculate last line of the edited code
        let l:newLastLine=search('</PrettyXML>')

        " Execute external formatter
        exe ":silent " . a:firstline . "," . l:newLastLine . "!xmllint --noblanks --format --recover -"

        " Recalculate first and last lines of the edited code
        let l:newFirstLine=search('<PrettyXML>')
        let l:newLastLine=search('</PrettyXML>')
    
        " Get inner range
        let l:innerFirstLine=l:newFirstLine+1
        let l:innerLastLine=l:newLastLine-1

        " Remove extra unnecessary indentation
        exe ":silent " . l:innerFirstLine . "," . l:innerLastLine "s/^  //e"

        " Remove fake tag
        exe l:newLastLine . "d" 
        exe l:newFirstLine . "d" 

        " Put the cursor at the first line of the edited code
        exe ":" . l:newFirstLine

        " Restore the file type
        exe "set ft=" . l:origft
endfunction
command! -range=% FormatXML <line1>,<line2>call DoFormatXML()

nmap <silent> <leader>x :%FormatXML<CR>
vmap <silent> <leader>x :FormatXML<CR>
````
## VRouter management webpage

You can use access the VRouter management webpage on port 8085

http://localhost:8085/

Here is some useful webpages.

### List of VRFs. 

http://localhost:8085/Snh_VrfListReq?name=

### You can click link in ucindex to see routes

http://localhost:8085/Snh_Inet4UcRouteReq?x=8

### Sandesh message stats

http://localhost:8085/Snh_SandeshMessageStatsReq?

## Check Cassandra

You can use this script

https://gist.github.com/nati/8064561

```
ubuntu@contrail-01:~$ python cas.py virtual-network
default-domain:default-project:__link_local__ 3e072e3c-59a9-402e-b61f-3697b81a7274
default-domain:default-project:default-virtual-network 3795cb36-1338-4292-936a-5541fbd06ee6
default-domain:default-project:ip-fabric 8adc0fdd-674f-44cd-ac1b-87854dd0ec87
default-domain:demo:right 6884a3b2-cfbe-4fff-88bb-52ef36146e0e
```

## Discovery Service

Check ip for clients

curl "http://node_ip:5998/clients.json" | python -mjson.tool

## Config daemon connection status

* Api Server

    `http://<config-node-ip>:8084/Snh_SandeshUVECacheReq?x=NodeStatus`

* Schema Transformer

    `http://<config-node-ip>:8087/Snh_SandeshUVECacheReq?x=NodeStatus`