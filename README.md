using System.Collections.Generic;
using System.Linq;
using System.Windows;
using System.Windows.Input;
using OSDIconFlashMap.Model;

namespace OSDIconFlashMap.View
{
    public partial class ImagePickerWindow : Window
    {
        private readonly List<ImageOption> _all;
        public ImageOption Selected { get; private set; }

        public ImagePickerWindow(IEnumerable<ImageOption> options)
        {
            InitializeComponent();
            _all = options?.ToList() ?? new List<ImageOption>();
            ic.ItemsSource = _all;
        }

        private void OnPick(object sender, MouseButtonEventArgs e)
        {
            if ((sender as FrameworkElement)?.DataContext is ImageOption op)
            {
                Selected = op;
                DialogResult = true;
                Close();
            }
        }
    }
}