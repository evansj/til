Here is a useful tip for extracting a file from a zip to stdout, so you can pipe it to `less` or whatever.

The `-p` flag will extract the contents of the given file(s) to stdout (-p = pipe), and the -c flag will do the same but also include some metadata about the file (-c = CRT).

Usage:

    unzip -p file.zip some/file/in/the/zip.txt

This is handy for finding out what version of a war file has been deployed, e.g.

    unzip -p something.war META-INF/maven/my.group.id/my.artifact.id/pom.xml |less
