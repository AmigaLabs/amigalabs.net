---
title: c# code to create shared library C code
---

To create automatically C code to be used into shared library version of clib4 you can use a small c# piece of code:

```c#
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Text.RegularExpressions;
using System.Threading.Tasks;

namespace clib4
{
    class Program
    {
        static StreamWriter file;
        static StreamWriter file1;
        static StreamWriter file3;
        static int stub = 112;

        static void Main(string[] args)
        {
            string mainDir = @"D:\Sviluppo\Amiga\clib4\library\include";
            file = File.CreateText(@"D:\vectors.txt");
            file1 = File.CreateText(@"D:\interface.txt");
            file3 = File.CreateText(@"D:\stubs.txt");
            Directory.Delete(@"D:\stubs", true);
            Directory.CreateDirectory(@"D:\stubs");

            if (File.Exists(mainDir))
            {
                // This path is a file
                ProcessFile(mainDir);
            }
            else if (Directory.Exists(mainDir))
            {
                // This path is a directory
                ProcessDirectory(mainDir);
            }
            else
            {
                Console.WriteLine("{0} is not a valid file or directory.", mainDir);
            }
            file.Close();
            file1.Close();
            file3.Close();
            Console.ReadKey();
        }

        // Process all files in the directory passed in, recurse on any directories 
        // that are found, and process the files they contain.
        public static void ProcessDirectory(string targetDirectory)
        {
            // Process the list of files found in the directory.
            string[] fileEntries = Directory.GetFiles(targetDirectory);
            foreach (string fileName in fileEntries)
                ProcessFile(fileName);

            // Recurse into subdirectories of this directory.
            string[] subdirectoryEntries = Directory.GetDirectories(targetDirectory);
            foreach (string subdirectory in subdirectoryEntries)
                ProcessDirectory(subdirectory);
        }

        // Insert logic for processing found files here.
        public static void ProcessFile(string path)
        {
            if (path.EndsWith("\\dos.h") || path.Contains("clib4_io.h"))
                return;
            if (path.EndsWith(".h"))
            {
                Console.WriteLine("Processed file '{0}'.", path);
                var lines = File.ReadAllLines(path);
                var function = "\t/* " + path.Replace("D:\\Sviluppo\\Amiga\\clib4\\library\\include\\", "").Replace("\\", "/") + " */\n";
                var signature1 = "\t/* " + path.Replace("D:\\Sviluppo\\Amiga\\clib4\\library\\include\\", "").Replace("\\", "/") + " */\n";
                int functionsFound = 0;
                foreach (var line in lines)
                {
                    if (line.StartsWith("extern"))
                    {
                        if (line.StartsWith("extern int profil("))
                            continue;

                        int end = line.IndexOf("(");
                        if (end >= 0)
                        {
                            function += "\t(void *) ";
                            int start = end - 1;
                            bool found = false;
                            while (!found)
                            {
                                var c = line.ElementAt(start);
                                if ((c == ' ' || c == '*') && (end - start > 1))
                                    found = true;
                                else
                                    start--;
                            }
                            stub += 4;
                            var functionName = line.Substring(start, end - start);
                            int linelen = ("(" + functionName.Trim().Replace("*", "") + "),").Length;

                            function += "(" + functionName.Trim().Replace("*", "") + ")," + string.Concat(Enumerable.Repeat(" ", 40 - linelen)) + " /* " + stub + " */\n";
                            Console.WriteLine(function);
                            functionsFound++;
                            /* Interface */
                            functionName = functionName.Replace("*", "").Trim();
                            var pattern = String.Format(@"\b{0}\b", functionName);
                            Match mtc = Regex.Match(line, pattern);
                            start = line.IndexOf("extern ") + 7;
                            var type = line.Substring(start, mtc.Index - 7);
                            var sign = line.Substring(mtc.Index + functionName.Length).Trim();

                            signature1 += "\t" + (type.EndsWith("*") ? type + " " : type) + "(* " + functionName + ") (" + (sign.StartsWith("(") ? sign.Substring(1) : sign);
                            linelen = ("\t" + (type.EndsWith("*") ? type + " " : type) + "(* " + functionName + ") (" + (sign.StartsWith("(") ? sign.Substring(1) : sign)).Length;
                            signature1 += string.Concat(Enumerable.Repeat(" ", 145 - linelen)) + " /* " + stub + " */\n";
                            /* Create Stub */
                            linelen = ("Clib4Call(" + functionName + ", " + stub + ");").Length;
                            file3.WriteLine("Clib4Call(" + functionName + ", " + stub + ");");
                        }
                        else
                            Console.WriteLine("Skipped line '{0}'.", line);
                    }
                }
                if (functionsFound > 0)
                {
                    function = function.Remove(function.Length - 1, 1);
                    file.WriteLine(function);
                    file1.WriteLine(signature1);
                }
            }
            else
                Console.WriteLine("Skipped file '{0}'.", path);
        }
    }
}
```

`mainDir` is the directory where clib4 source code is. `file`, `file1 `and `file3 `contain respectively vectors, interface and stubs calls.
Chnage the file path where you want to create these files.  


Then you can replace:
* The content of `shared_library/clib4_vectors.h` in `static void *main_vectors[]` array starting after latest `(void *) (LIB_Reserved),` with `file` content.
* The content of `shared_library/interface.h` in `struct Clib4IFace` starting after `void (* internal5)(void);  //116` with `file1` content.
* The content of `shared_library/stubs.c` starting after `/* Script created stubs */` with file3 content.


The process is possible thanks to `extern` calls present into clib4 include files