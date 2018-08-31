# dll ���� Embedding

����� Fody�� ����ϹǷ� �� ����� ������� �ʴ´�.

``` c#
static Assembly ResolveAssembly(object sender, ResolveEventArgs args)
{
    Assembly thisAssembly = Assembly.GetExecutingAssembly();
    var name = args.Name.Substring(0, args.Name.IndexOf(',')) + ".dll";

    string[] names = thisAssembly.GetManifestResourceNames();
    var resources = thisAssembly.GetManifestResourceNames().Where(s => s.EndsWith(name));
    if (resources.Count() > 0)
    {
        string resourceName = resources.First();
        using (Stream stream = thisAssembly.GetManifestResourceStream(resourceName))
        {
            if (stream != null)
            {
                byte[] assembly = new byte[stream.Length];
                stream.Read(assembly, 0, assembly.Length);
                return Assembly.Load(assembly);
            }
        }
    }
    return null;
}
```

���ν����� ȣ����(`Program.cs:Main` �Լ�)�� �� �ڵ带 �߰��Ѵ�.

``` c#
AppDomain.CurrentDomain.AssemblyResolve += new ResolveEventHandler(ResolveAssembly);
```

���� ���ҽ��� dll ������ �ְ� `FileType`�� `Binary`�� �ٲٸ� �ȴ�. ���� dll ������ �������Ͽ� ���Ե� ���̴�.