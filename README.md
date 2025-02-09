
------------------------------------Дизайн---------------------------------
<Window x:Class="LabWork16.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:LabWork16"
        
        mc:Ignorable="d"
        Title="MainWindow" Height="450" Width="800" Loaded="Window_Loaded">
    <DockPanel>
        <Menu DockPanel.Dock="Top">
            <Menu.ItemsPanel>
                <ItemsPanelTemplate>
                    <DockPanel HorizontalAlignment="Right"/>
                </ItemsPanelTemplate>
            </Menu.ItemsPanel>
            <MenuItem Header="Запустить новую задачу"/>
            <Separator/>
            <MenuItem Name="Leave" Header="Выход" Click="LeaveItem_Click" Height="25"/>
        </Menu>
        <ToolBar DockPanel.Dock="Top">
            <ToolBar.Resources>
                <Style TargetType="{x:Type ToolBarPanel}">
                    <Setter Property="Orientation" Value="Vertical"/>
                </Style>
            </ToolBar.Resources>

            <DockPanel>
                <Button x:Name="RefreshButton" Content="Обновить" DockPanel.Dock="Right" HorizontalAlignment="Right" />
                <Separator/>
                <Button x:Name="KillTaskButton" Content="Снять задачу" Click="KillTaskButton_Click" DockPanel.Dock="Right" HorizontalAlignment="Right"/>
                <Separator/>
                <Button x:Name="KillTasksButton" Content="Завершить дерево процессов" DockPanel.Dock="Right" HorizontalAlignment="Right"/>
            </DockPanel>
        </ToolBar>
        <TabControl TabStripPlacement="Left">
            <TabItem Header="Процессы" Height="40">
                <ListBox>
                    <ListBox.ContextMenu>
                        <ContextMenu>
                            <MenuItem Header="Снять задачу"/>
                            <MenuItem Header="Завершить дерево процессов"/>
                        </ContextMenu>
                    </ListBox.ContextMenu>
                </ListBox>
            </TabItem>
            <TabItem Header="Приложения" Height="40"/>
        </TabControl>
    </DockPanel>
</Window>
-------------------------------------------------Код-------------------------------------------------------------
 private const int GWL_STYLE = -16;
 private const int WS_SYSMENU = 0x80000;

 [System.Runtime.InteropServices.DllImport("user32.dll", SetLastError = true)}
 private static extern int GetWindowLong(System.IntPtr hWnd, int nIndex);

 [System.Runtime.InteropServices.DllImport("user32.dll")}
 private static extern int SetWindowLong(System.IntPtr hWnd, int nIndex, int dwNewLong);
 public MainWindow()
 {
     InitializeComponent();
     Loaded += Window_Loaded;
 }

 private void KillTaskButton_Click(object sender, RoutedEventArgs e)
 {

 }

 private void Window_Loaded(object sender, RoutedEventArgs e)
 {
     var hwnd = new System.Windows.Interop.WindowInteropHelper(this).Handle;
     SetWindowLong(hwnd, GWL_STYLE, GetWindowLong(hwnd, GWL_STYLE) & ~WS_SYSMENU);
 }

 private void LeaveItem_Click(object sender, RoutedEventArgs e)
 {
     Close();
 }
