using System;
using System.Windows;
using System.Windows.Controls;

namespace NovaCID
{
    public partial class NovaCIDView : UserControl
    {
        public NovaCIDView()
        {
            InitializeComponent();
            UpdateLeftKnobPosition(LeftKnob_SliderX.Value, LeftKnob_SliderY.Value);
        }

        private void LeftKnob_Slider_ValueChanged(object sender, RoutedPropertyChangedEventArgs<double> e)
        {
            if (LeftKnob_TextX != null && LeftKnob_TextY != null)
            {
                double x = LeftKnob_SliderX.Value;
                double y = LeftKnob_SliderY.Value;

                LeftKnob_TextX.Text = Math.Round(x).ToString();
                LeftKnob_TextY.Text = Math.Round(y).ToString();

                UpdateLeftKnobPosition(x, y);
            }
        }

        private void LeftKnob_TextX_TextChanged(object sender, TextChangedEventArgs e)
        {
            if (double.TryParse(LeftKnob_TextX.Text, out double value))
            {
                value = Math.Max(0, Math.Min(2560, value));
                LeftKnob_SliderX.Value = value;

                double y = LeftKnob_SliderY.Value;
                UpdateLeftKnobPosition(value, y);
            }
        }

        private void LeftKnob_TextY_TextChanged(object sender, TextChangedEventArgs e)
        {
            if (double.TryParse(LeftKnob_TextY.Text, out double value))
            {
                value = Math.Max(0, Math.Min(1440, value));
                LeftKnob_SliderY.Value = value;

                double x = LeftKnob_SliderX.Value;
                UpdateLeftKnobPosition(x, value);
            }
        }

        private void UpdateLeftKnobPosition(double centerX, double centerY)
        {
            double outerRadius = 186;
            double innerRadius = 112;

            Canvas.SetLeft(LeftKnobOuter, centerX - outerRadius);
            Canvas.SetTop(LeftKnobOuter, centerY - outerRadius);

            Canvas.SetLeft(LeftKnobInner, centerX - innerRadius);
            Canvas.SetTop(LeftKnobInner, centerY - innerRadius);

            Canvas.SetLeft(LeftKnobCenterPoint, centerX - 10);
            Canvas.SetTop(LeftKnobCenterPoint, centerY - 10);
        }
    }
}